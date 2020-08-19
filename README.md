# Creating beautiful print formats in ERPNext.

ERPNext is a beautiful software and many organizations use it. And print formats are really important to organizations as hardcopy of every document with their letterhead is a core requirement. Recreating their template in ERPNext is an art and not easy.

I’ll break down the process to make it easy for all the beginners.

ERPNext uses Jinja templating for this purpose. I’m not going to give you a deep explanatory course on Jinja, but the basics that you’ll need for a neat, aesthetic template.

When the client gives you their existing template, you should make a thorough observation of it.

## First off, we split the template into 3 parts: Above, Middle and Below.
1. Above: The above part includes all the static items in the page including the letter head, the customer details and the head of the table(if any).

2. Middle: This part consists of recurring things in the template. Eg: the rows in the table.

3. Bottom: This part consists of static items in the bottom half of the template, including the footer of the table, other calculations and footer of the template.

Now that we have divided the template into three parts. It becomes fairly easier for us to do things and also makes the code look beautiful(you’ll understand how as you keep reading).
We will now use macros for the above and bottom sections/parts. Macros are pieces of code defined by a name. And hence, you can call them just by that name whenever you need them(like functions).

## Define a macro:
```
{% macro name_of_macro() %}
	<p>Hi</p>
{% endmacro %}
```

Now, you can refer to that macro anywhere(below its declaration) with:
```{{ name_of_macro() }}```

Each time you call it, the code inside of it replaces it.
```
	<p>Test</p>			>		<p>Test</p> 
	{{ name_of_macro() }}		>		<p>Hi</p>
	<p>Again test</p>		>		<p>Again test</p>
	{{ name_of_macro() }}		>		<p>Hi</p>
	<p>Finish</p>			>		<p>Finish</p>
```

## Recreate the the whole template in html with the help of bootstrap.
Here is a basic example of HTML code with bootstrap:
```
<div class="container-fluid">
	<div class="row d-flex align-items-end justify-content-between" style="padding-top: 10px; font-size: 13px">
        	<div class="col-xs-12">
            		Dear Sir/Madam,<br>
            		Thank you for placing the purchase order. We hearby confirm our acceptance as follows:
        	</div>
    	</div>
</div>
```
Using **col-xs** is necessary as anything else will cause wkhtmltopdf to break and the the generated PDF will loose its alignment, although the print version will look fine. **Intendation** is also very important as if increases the readablity of the code and when working as a team, it gives a great advantage.

Now that you have converted the basic template into html format, its time for us to use Jinja to fetch value into our format.
The format for it is ```{{ name of variable to print }}```

In ERPNext, these variables are inside each doctypes. And to access them, we use doc.name_of_field.
To get the fieldname of a column, we can go to customise form and select the doctype we want to and search for the field and get its fieldname.
Example code to fetch document name: ```{{ doc.name }}```

## Split to macros
Now that we know how to fetch values and that we have the template in HTML, we can actually split the template into macros.

### Above Section
```
{% macro above_items() %}
<div class="container-fluid" style="min-width: 100% !important; min-height: 210mm !important;">
    <div class="row d-flex align-items-end justify-content-end">
        <div class="col-xs-4 text-left" style="margin-top: -15px">
            <img alt="Logo" width=100% src="/files/nameoflogo.png">
        </div>
        <div class="col-xs-8 text-right">
            <b>SALES ORDER</b>
        </div>
    </div>
    
    <div class="row d-flex align-items-end justify-content-between" style="padding-top: 10px; font-size: 13px">
        <div class="col-xs-6">
            <b>{{ doc.customer_name }}</b><br>
            {{ doc.shipping_address }}
        </div>
        <div class="col-xs-6 text-right">
            <span style="font-size: 13px">
                <b>{{ doc.name }}</b><br>
            </span>
            {{ doc.transaction_date }}<br>
            <b>Your PO Reference</b><br>
            {{ doc.po_no }}
        </div>
    </div>
    
    <div class="row" style="padding-top: 10px; font-size: 13px">
        <div class="col-xs-12">
            Dear Sir/Madam,<br>
            Thank you for placing the purchase order. We hearby confirm our acceptance as follows:
        </div>
    </div>
	<table class="table table-bordered text-wrap" style="overflow-x: hidden; table-layout: fixed !important">
        <thead>
            <tr class="table-info" style="font-weight: bold;">
                <td style="width:6%;">Sr</td>
                <td>Product</td>
                <td style="width:6%;">Unit</td>
                <td style="width:10%;">Qty</td>
                <td style="width:22%;">Order</td>
            </tr> 
        </thead>
        <tbody>
{% endmacro %}
```
Notice how the table is started in the above part, but not closed. It's because we assign the body of the table to the middle part/section and the footer to the below.

### Below Section
Once we create the above section, it is time for us to create the below section. We will move on to the middle section after the below part.
```
{% macro below_items() %}
        </tbody>
        <tfoot style="font-weight: bold;">
            <tr class="table-info">
                <td colspan='3' style="text-align: right">Total Quantity</td>
                <td>{{ "%.2f"|format(doc.total_qty|float) }}</td>
                <td></td>
            </tr>
        </tfoot>
    </table>
</div>
<!-- Our first container was closed here. It has height of 210mm -->

<div class="container-fluid" style="min-width: 100% !important;">
    <div class="row d-flex align-items-end justify-content-between" style="font-size: 13px;">
        <div class="col-xs-6">
            Store Incharge:<br><br>
            Authorised Signatory:
        </div>
        <div class="col-xs-6">
            Signature:<br><br>
            Signature:
        </div>
    </div>
</div>
{% endmacro %}
```
Now, if we check the below part, we can see how the table we opened in the above section was beautifully closed. See how the intendation makes it easier to read the code!
Now, if we just print the above and below sections, we will be able to see the template(without any rows in the table ofc!)

### Middle Section
Now comes the best part! The middle section, where all the fun is!
Here, we don't define it as a seperate macro, rather, we just write it down(You're always free to write it as a macro and call as well!)
```
{{ above_items() }}
{% set pr = [1] %}
{% set lines = [0] %}
    {%- for row in doc.items -%}
        {% if lines[-1] %} {% endif %} 
        {% if lines.append( lines[-1] + 2 +((row.description|length / 38)|int)) %}{% endif %}
        {% if (row.description|length % 38) > 0 %}
            {% if lines[-1] %} {% endif %} 
            {% if lines.append( lines[-1] + 1 ) %}{% endif %}
        {% endif %}
        {% if (lines[-1]/30) <= pr[-1] %}
            <tr class="table-info">
                <td>{{ row.idx }}</td>
                <td>{{ row.item_name }}<br>{{ row.serial_no }}</td>
                <td>{{ row.uom }}</td>
                <td>{{ row.qty }}</td>
                <td>{{ row.purchase_order }}</td>
            </tr>
        {% else %}
            {{ below_items() }}
            <div style="page-break-before: always;"></div>
            {% if pr[-1] %} {% endif %} 
            {% if pr.append( pr[-1] + 1 ) %}{% endif %}
            {{ above_items() }}
            <tr class="table-info">
                <td>{{ row.idx }}</td>
                <td>{{ row.item_name }}<br>{{ row.serial_no }}</td>
                <td>{{ row.uom }}</td>
                <td>{{ row.qty }}</td>
                <td>{{ row.against_sales_order }}</td>
            </tr>
        {% endif %}
    {%- endfor -%}
{{ below_items() }}
```

This middle section first calls the above_items(). This places all the above items in our print.
Then we define two lists in Jinja. One for the number of lines our table has, and another for number of pages required.

I calculated the width length of my product description column and understood that it can hold upto 38 characters without breaking into a new line.

Then we run a loop through the table and add 2(which is a fixed number because rows themselves takes up some space) plus length of the description divided by 38(after converting it to int). Then we check if there is float(decimal) value upon that division, and if any, we add 1 to it. And thus, we get the number of lines required by the rows in that table.

Now, I did my experiments and found out that I can have no more than 30 lines in 1 page. So we have to break the page after every 30 lines.
So we do a simple check before printing the row to the page if it will exceed the maximum number of lines allowed. We do that by dividing it by 30 and checking if it is less or equal to the number of pages required till now.

If it is less or equal, we print the row. And if not, we call for the below items which will put the footer and other items specified in it. And then, we break the page.
Once we break the page, we increase the number of pages required and call for the above items since we moved on to new page and print the current row there.

This loop continues until the last row after which a below_items() will be called which will end the last page.

And there you have, beautifully printed dynamic invoices and receipts!
Happy coding and printing!
