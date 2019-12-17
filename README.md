# Flash Builder Integration

This repository was created for demonstration purposes on how Digital Apparel Lab Flash builder can be integrated with e-commerce web site.

# Quick Start Example

1. Obtain Flash Builder end point URL from Digital Apparel Lab
2. Put provided end point URL in index.html and addToCart.html files in place of #to_be_provided# place holder.
3. Host all files from this repository on any web server (or locally), for example on Visual Studio Code with Live Server Extension.
4. Open index.html

# Integration basics

Here are the steps for integrating with Flash builder

1. Create Flash builder session, specifying: 
   - session id, 
   - success redirect url for adding configured products to e-commerce shopping cart, 
   - cancel redirect url for cancelling the configuration session
2. Launch Flash builder with pre-created session id as a startup parameter
3. Receive call back from Flash builder 
   - Option 1, when configuration session is sucessfully finished (add to cart redirect )
     - extract configured products from Flash builder session
     - merge configured products with e-commerce shopping cart
   - Option 2, when configuration session is canceled 
     - redirect to e-commerce landing page dedicated for it
4. Re-open Flash builder for making changes

# Create Flash builder session

1. **Generate Flash builder session id**

   E-commerce web site should generate good unique session id.  This session id should be saved for launching Flash builder later. 

   JavaScript example:
~~~
  function uuid() {
      return ([1e7]+-1e3+-4e3+-8e3+-1e11).replace(/[018]/g, c =>
          (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
      );
~~~
2. **Create session object for passing to Create Session end point**
   
   Session object contains the following parameters:
  
   - **sessionVariable**: Flash builder session id,
   - **redirectURL**: redirect URL for returning back to e-commerce web site and adding configured products to the shopping cart. This URL could contain extra parameters for defining e-commerce shopping cart session, which will be passed back as defined.  
   - **redirectURLCancel**: redirect URL for returning back to e-commerce web site and cancelling the configuration session. This URL also could contain extra parameters for defining e-commerce shopping cart session, which will be passed back as defined.
   
   JSON example: 
~~~
  { 
   "sessionVariable":"Flash builder session id",
   "redirectURL":"addToCartUrl",
   "redirectURLCancel":"cancelUrl"
  }
~~~

3. **Call Create Session end point**

  Perform a POST call to Create Session end point for informing Flash builder about session paramaters.

~~~
  $.ajax({
    type: "POST",
    url: endPointURL + "/Services/SCGATGService.asmx/CreateSession",
    data: JSON.stringify(sessionObject),
    cache: false,
    contentType: "application/json; charset=utf-8",
    dataType: "json",
    success: function (response) {
     // launch Flash builder here
    },
    error: function (response, status, error) {
        // process exception here
    }
  });
~~~
  
# Launch Flash builder with pre-created session id as a startup parameter

Flash builder can be launched by the following URL once session is sucessfully created.

~~~
  endPointURL + "/Pages/SCGProductPage.aspx?sessionVariable=" + flashBuilderSessionId
~~~

Our standard way is to launch it in 740x1000 HTML iFrame:

~~~
  <iframe id="flashIframe" height="740" width="1000" frameborder="0" scrolling="no" allowtransparency="true" src_= "endPointURL/Pages/SCGProductPage.aspx?sessionVariable=flashBuilderSessionId" />
~~~

# Receive call back from Flash builder 

All call backs from Flash builder are appended with Flash builder **sessionVariable** URL parameter for passing it back to e-commerce web site.

**Scenario 1: Call back to redirectURL when Add To Cart button is clicked on Flash builder**

   1. Extract Flash builder **sessionVariable** URL parameter
   2. Prepare close session object for calling Close Session end point.
   
      Close session object contains the following parameter:  
      - **sessionVariable**: Flash builder session id
      
      JSON example: 
~~~
{ 
  "sessionVariable":"Flash builder session id",
}
~~~     
    
   3. Call Close Session end point and receive Flash builder session data

   Perform a POST call to Close Session end point for receiving product data from Flash builder
   
~~~
  $.ajax({
      type: "POST",
      url: endPointURL + "/Services/SCGATGService.asmx/CloseSession",
      data: JSON.stringify(sessionObject),
      cache: false,
      contentType: "application/json; charset=utf-8",
      dataType: "json",
      success: function (response) {
         // process received data here
          var data = JSON.parse(response.hasOwnProperty('d') ? response.d : response);
      },
      error: function (response, status, error) {
         // process exception here
      }
  });
~~~    

   Close Session end point respons with the JSON object defining:
   - SaveCode: defines the configured products and needs to be communicated to Digital Apparel Lab for manufacturing the order
   - TotalQty
   - TotalAmount
   - oldSaveCode: old save code the Flash builder session was re-opened from (defined only when session is re-opened)
    
   Example: 
~~~
{ 
   "SaveCode":"FLA1234",
   "TotalQty":12,
   "TotalAmount":123.45,
   "oldSaveCode":"FLA4321"
}    
~~~
   
   4. Merge received data from Flash builder with e-commerce shopping cart
   
**Scenario 2: Call back to redirectURLCancel when Cancel button is clicked on Flash builder**

   E-commerce web site should process this call back internally.
   
# Re-open Flash builder for making changes

Flash builder supports a special launch mode for making changes to configured products. You need to retain Flash builder session id for  closed Flash builder session and re-open Flash builder with that session specified. The process of re-opening the Flash builder is exactly the same like launching it with new session id. See **Launch Flash builder with pre-created session id as a startup parameter** for details. **Note**: all changes to the configured products will be referenced to new **Save Code** returned by Close Session end point, and **Old Save Code** will be returned for e-commerce web site reference.
