Lab 4.2: Modify AS3 Declaration using BIG-IQ 6.1
------------------------------------------------

Using the declarative AS3 API, let's modfiy the HTTP application created during the previous **lab 1 - Task 1** through BIG-IQ using an updated AS3 declaration.

Task 5 - Add an HTTPS Application to existing HTTP AS3 Declaration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: Start with the previous AS3 Declaration from **lab 1 - Task 1**

#. Copy below example of an AS3 Declaration into the AS3 public validator.

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 7,21,22,38,41

   "MyWebApp6": {
           "class": "Application",
           "template": "https",
           "statsProfile": {
               "class": "Analytics_Profile",
               "collectedStatsInternalLogging": true,
               "collectedStatsExternalLogging": false,
               "capturedTrafficInternalLogging": false,
               "capturedTrafficExternalLogging": true,
               "collectPageLoadTime": true,
               "collectClientSideStatistics": true,
               "collectResponseCode": true,
               "sessionCookieSecurity": "ssl-only"
           },
           "serviceMain": {
               "class": "Service_HTTPS",
               "virtualAddresses": [
                   "<virtual>"
               ],
               "pool": "web_pool",
               "profileAnalytics": {
                   "use": "statsProfile"
               },
               "serverTLS": "webtls"
           },
           "web_pool": {
               "class": "Pool",
               "monitors": [
                   "http"
               ],
               "members": [
                   {
                       "servicePort": 80,
                       "serverAddresses": [
                           "<node3>",
                           "<node4>"
                       ],
                       "shareNodes": true
                   }
               ]
           },
           "webtls": {
               "class": "TLS_Server",
               "certificates": [
                   {
                       "certificate": "webcert"
                   }
               ]
           },
           "webcert": {
               "class": "Certificate",
               "certificate": {
                   "bigip": "<cert>"
               },
               "privateKey": {
                   "bigip": "<key>"
               }
           }
       }

To access to the AS3 public validator, go to the Linux Jumphost, open a browser and connect to http://10.1.1.14:5000

#. Click on ``Format JSON`` on the top left.

#. Click on ``Validate JSON`` and ``Validate AS3 Declaration``. Make sure the Declaration is valid!

#. Now that the JSON is validated, let's add the targetHost (BIG-IQ) and the traget (BIG-IP device)

Modify the Virtual Address to 10.1.20.104 and the serverAddresses from 10.1.10.100 to 10.1.10.104.

#. Click on  ``Format JSON``, ``Validate JSON`` and ``Validate AS3 Declaration``. Make sure the Declaration is valid!

|lab-2-1|

#. Using Postman, use the **BIG-IQ AS3 Declaration** collection in order to create the service on the BIG-IP through BIG-IQ. Copy/Past the declaration into Postman.

POST https://10.1.1.4/mgmt/shared/appsvcs/declare

Use the **BIG-IQ Check AS3 deployment** collection to ensure that the AS3 deployment is successfull without errors: 

GET https://10.1.1.4/mgmt/cm/global/tasks/deploy-app-service


#. Logon on BIG-IQ as admin, go to Application tab and check the application is displayed and analytics are showing.


.. |lab-2-1| image:: images/lab-2-1.png
   :scale: 80%
