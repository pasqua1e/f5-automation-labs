Lab 4.3: Deploying AS3 Templates with BIG-IQ 6.1
------------------------------------------------

Task 6 - Create custom HTTP AS3 Template on BIG-IQ
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Copy the below example of an AS3 service template into the Postman **BIG-IQ AS3 Template Creation** call.
It will create a new template in BIG-IQ AS3 Service Catalogue:

    POST https://10.1.1.4/mgmt/cm/global/appsvcs-templates

.. code-block:: yaml
   :linenos:

    {
        "description": "",
        "name": "HTTPcustomTemplateTask6",
        "schemaOverlay": {
            "type": "object",
            "properties": {
                "class": {},
                "updateMode": {},
                "schemaVersion": {},
                "id": {},
                "label": {},
                "remark": {},
                "constants": {},
                "Common": {},
                "controls": {},
                "scratch": {}
            },
            "required": [
                "class"
            ],
            "definitions": {
                "Pool": {
                    "loadBalancingMode": {
                        "type": "string",
                        "default": "round-robin"
                    },
                    "monitors": {
                        "type": "array"
                    },
                    "members": {
                        "type": "array"
                    }
                },
                "Service_HTTP": {
                    "virtualPort": {
                        "type": "integer",
                        "default": 8080,
                        "const": 8080
                    },
                    "virtualAddresses": {
                        "type": "array"
                    }
                }
            }
        }
    }


2. Logon on BIG-IQ, go to Application tab, then Application Templates. Look at the custom template created previous through the API.

|lab-3-1|

Note the AS3 Template cannot be created through BIG-IQ UI but only using the API. You can only delete a AS3 templates from the BIG-IQ UI.

You can see the Template in JSON format if you click on it.

|lab-3-2|


Task 7 - Admin set RBAC for Olivia on BIG-IQ
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's update now Oliva's service catalog.

Logon on BIG-IQ as admin go to the System tab, Role Management, Roles, CUSTOM ROLES, Application Roles, select **Applicator Creator AS3** 
and the custom role linked to the custom HTTP template previously created. Remove the **default** template from the allowed list. 
Click **Save & Close**.

|lab-3-3|


Task 8 - As Olivia, deploy the HTTP Application Service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Using Postman, update the user to olivia/olivia in the **BIG-IQ Token** call (body).

2. Copy below example of an AS3 Declaration into the body of the **BIG-IQ AS3 Declaration** collection in order to create the service on the BIG-IP through BIG-IQ:

POST https://10.1.1.4/mgmt/shared/appsvcs/declare?async=true


.. code-block:: yaml
   :linenos:

    {
        "class": "AS3",
        "action": "deploy",
        "declaration": {
            "class": "ADC",
            "target": {
                "hostname": "ip-10-1-1-10.us-west-2.compute.internal"
            },
            "schemaVersion": "3.7.0",
            "id": "isc-lab",
            "controls": {
                "class": "Controls",
                "logLevel": "debug"
            },
            "Task8": {
                "class": "Tenant",
                "A8": {
                    "class": "Application",
                    "schemaOverlay": "HTTPcustomTemplateTask6",
                    "template": "http",
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
                        "class": "Service_HTTP",
                        "virtualAddresses": [
                            "10.1.20.105"
                        ],
                        "pool": "pool_8",
                        "profileAnalytics": {
                            "use": "statsProfile"
                        }
                    },
                    "pool_8": {
                        "class": "Pool",
                        "monitors": [
                            "http"
                        ],
                        "members": [
                            {
                                "serverAddresses": [
                                    "10.1.10.111"
                                ],
                                "servicePort": 80
                            }
                        ]
                    }
                }
            }
        }
    }

  
This will give you an ID which you can query using the **BIG-IQ Check AS3 Deployment Task**.

3. Use the **BIG-IQ Check AS3 Deployment Task** and **BIG-IQ Check AS3 Deployment** Postman calls to ensure that the AS3 deployment is successfull without errors: 

   GET https://10.1.1.4/mgmt/shared/appsvcs/task/<id>
   
   GET https://10.1.1.4/mgmt/cm/global/tasks/deploy-app-service

4. Logon on **BIG-IP A** and verify the Application is correctly deployed in partition Task8.

5. Logon on **BIG-IQ** as Olivia, go to Application tab and check the application is displayed and analytics are showing.

|lab-3-4|


Task 9 - Delete Task1 and Task 2 AS3 Applications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use below AS3 declaration to delete couple of the application previoulsy created.

.. code-block:: yaml
   :linenos:

   {
       "class": "AS3",
       "action": "deploy",
       "persist": true,
       "declaration": {
           "class": "ADC",
           "schemaVersion": "3.7.0",
           "id": "example-declaration-01",
           "label": "Task1",
           "remark": "Task 1 - HTTP Application Service",
           "target": {
               "hostname": "ip-10-1-1-10.us-west-2.compute.internal"
           },
           "Task1": {
               "class": "Tenant"
           },
           "Task2": {
               "class": "Tenant"
           }
       }
   }

Here, we empty the tenants/partitions: Task1 and Task2. This should remove those partitions from BIG-IP A. The relevant Apps 
should also disappear from BIG-IQ. 

.. |lab-3-1| image:: images/lab-3-1.png
   :scale: 60%
.. |lab-3-2| image:: images/lab-3-2.png
   :scale: 60%
.. |lab-3-3| image:: images/lab-3-3.png
   :scale: 60%
.. |lab-3-4| image:: images/lab-3-4.png
   :scale: 60%
