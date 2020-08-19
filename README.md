# Creating beautiful print formats in ERPNext.

ERPNext is a beautiful software and many organizations use it. And print formats are really important to organizations as hardcopy of every document with their letterhead is a core requirement. Recreating their template in ERPNext is an art and not easy.

I’ll break down the process to make it easy for all the beginners.

ERPNext uses Jinja templating for this purpose. I’m not going to give you a deep explanatory course on Jinja, but the basics that you’ll need for a neat, aesthetic template.

When the client gives you their existing template, you should make a thorough observation of it.
First off, we split the template into 3 parts: Above, Middle and Below.
1. Above: The above part includes all the static items in the page including the letter head, the customer details and the head of the table(if any).

2. Middle: This part consists of recurring things in the template. Eg: the rows in the table.

3. Bottom: This part consists of static items in the bottom half of the template, including the footer of the table, other calculations and footer of the template.

Now that we have divided the template into three parts. It becomes fairly easier for us to do things and also makes the code look beautiful(you’ll understand how as you keep reading).
We will now use macros for the above and bottom sections/parts. Macros are pieces of code defined by a name. And hence, you can call them just by that name whenever you need them(like functions).
Here is how you define a macro:
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

Now, we recreate the the whole template in html with the help of bootstrap.
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

Now that we know how to fetch values and that we have the template in HTML, we can actually split them.
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
	<table class="table table-bordered text-wrap" style="overflow-x: hidden; table-layout: fixed; font-size:13px !important">
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
