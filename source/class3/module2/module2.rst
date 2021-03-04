Module 2: Check how Device ID+ works
####################################

In this module, we will work with Device ID+ and get the identifier reported back into /var/ltm/log of BIG-IP.
Additional a BIG-IP reports into ELK Stack. So you can correlate data for anaylses.

.. warning:: This is not a full integration to show a Device ID+ / AWF Demo. Currently we are working on Use Cases which can be integrated into the Demo.

.. note:: We are currently working on a use case to leverage Device ID+ together with AWF and will update the Module accordingly. If you got a specific AWF "Device ID+ Usecase" in mind, please reach out to Patrick Zoller.

**Device ID+ Overview**

DeviceID+ is a pseudonymized identifier that can be used to identify a device visiting a customer's applications. 
More precisely, it is an identifier for a browser or a native mobile application. For the initial release of DeviceID+, we are only focusing on Web and Mobile Web applications.

.. note:: If you haven´t worked with Device ID+ before, please review the `Device ID+`_ Article on F5 Cloud Services.

.. _`Device ID+` : https://f5cloudservices.zendesk.com/hc/en-us/categories/360005886653-Device-ID-


**Check how Device ID+ works**

    #.  Connect to BIG-IP named "BIG-IP 16.0 generic demos and Device ID+" via TMUI.

        .. image:: ../pictures/module2/img_class3_module2_animated_1.gif
           :align: center
           :scale: 30%
    
    #. Within the WebUI of the BIG-IP instances navigate to iApps › Application Services : Applications › deviceID and select `Reconfigure`.

        .. image:: ../pictures/module2/img_class3_module2_animated_2.gif
           :align: center
           :scale: 30%

    #. Within the iApp configuration you will find predefined JS Injection configuration in the `1JS` part. Furthermore the 1JS gets been injected on the Virtual Server named `arcadia.emea.f5se.com_vs`.
       We leave the rest of the configuration untouched. 

        .. image:: ../pictures/module2/img_class3_module2_animated_3.gif
           :align: center
           :scale: 30%

.. note::  F5 Cloud Services on `Getting Started with F5 Device ID+`_ cover the application onboard with F5 Device ID+ on BIG-IP in more detail.

.. _`Getting Started with F5 Device ID+` : https://f5cloudservices.zendesk.com/hc/en-us/articles/360060301673-Getting-Started-with-F5-Device-ID-


**Device ID+ and iRule**

Device ID+ includes two identifiers – a residue-based identifier and an attribute-based identifier. The residue-based identifier is based on local storage and cookies. 
The attribute-based identifier is based on signals collected on the device. The two identifiers always have different values.

1JS writes both the residue-based and attribute-based identifiers in a single, first-party cookie called *_imp_apg_r_*. The *_imp_apg_r_* cookie is URL encoded with the following format:

%7B%22diA%22%3A%22AT9cyV8AAAAAd60uXCtYafPTZGLaVAku%22%2C%22diB%22%3A%22ASJ4gFmzPo%2Fa8AHJceWhykudRoXeBGlP%22%7D

This cookie can be decoded via https://www.urldecoder.org/ to get the response in clear text. The decoded cookie has the following format:

.. code-block::


    "diA": "AT9cyV8AAAAAd60uXCtYafPTZGLaVAku"
    "diB": "ASJ4gFmzPo/a8AHJceWhykudRoXeBGlP"


.. note:: Here, diA represents the residue-based identifier and diB represents the attribute-based identifier.

**How to decode Device ID+ _imp_apg_r_ cookie with an iRule**

    #. Within BIG-IP we use an iRule named *print_deviceid* and do a URL decoding of the *_imp_apg_r_* cookie and log *diA* and *diB* into /var/log/ltm of BIG-IP.
    
    #. The irule named *print_deviceid* has been attached to Virtual Server named `arcadia.emea.f5se.com_vs`.

        .. image:: ../pictures/module2/img_class3_module2_animated_4.gif
           :align: center
           :scale: 30%
 
**How to test Device ID+**

    #. To verify and view the logged values, connect to BIG-IP named "BIG-IP 16.0 generic demos and Device ID+" via SSH. 
    #. Run *run util bash* followed by *tail -f /var/log/ltm* in the SSH Session.
    #. RDP to windows machine called *win-client*.
    #. Launch Chrome.
    #. Open Devtools (Keyboard F12), select XHR in the Devtools and select the Browser Tab named *Device ID check*.
    #. Check the request and response in Chrome.
    #. Also check the cookie on the Devtools under Application.

         .. image:: ../pictures/module2/img_class3_module2_animated_5.gif
           :align: center
           :scale: 30%


    #. You may want to do further test by running `Chrome`in Incognito Modus and compare the values of `diA` and `diB` with the outcome of the previous test.
    #. Also check *tail -f /var/log/ltm* in the SSH Session as the values of `diA` and `diB` of the *_imp_apg_r_* cookie have been written to the file.

        .. image:: ../pictures/module2/img_class3_module2_animated_6.gif
           :align: center
           :scale: 30%


**Device ID+ and ELK**

Within the UDF Environment you will find an instance called **ELK**.
Here we run an ELK Container which is used to visualize Device Identifier and correlate data i.e. Username to Device ID; Geo IP to Device ID.

.. note:: This is a MVP. So please reach out if you have use cases which we should add to the Demo.

Steps: 
RDP to windows machine called *win-client*. The Password of the instance is listed within the **Details / Documentation** Tab.

    #. Launch Chrome and choose the bookmark called ``Device ID+ Kibana``.
    #. Klick the button left to "Home". Within the Kibana Section you can choose between **Discover** or **Dashboard**.
 
        .. image:: ../pictures/module2/img_class3_module2_animated_7.gif
           :align: center
           :scale: 30%

**Demo some Use Cases - Deliberate use of proxy networks**

Within that use case we will cover a sudden fluctuations in IPs per DeviceID.
As the Source IP of the RDP Client is static, we will use XFF to simulate access from the same browser, but changing the Source IP Address.

Steps:
    #. Launch Chrome and discover the browser add-on called **IP**.
    #. With the help of the add-on, we can simulate access by the same browser but leverage different IP address.
    #. Feel free to use whatever IP address comes up to your mind. A list with example IP addresses is stored on clients desktop.

        .. image:: ../pictures/module2/img_class3_module2_animated_8.gif
           :align: center
           :scale: 30%

    #. Select one IP Address and access the bookmark called **Device ID check**.
    #. With that you will access the **Arcadia Application**.
    #. Go back to **Device ID+ Kibana**, select **Discover** and filter for **deviceA** within the variable fields.
  
        .. image:: ../pictures/module2/img_class3_module2_animated_9.gif
           :align: center
           :scale: 30%

    #. Note the IP Address which has been collected by hover over the field **xff_ip**.
    #. Back to the add-on called **IP** select another IP address of your choice.
    #. Issue another request to the **Arcadia Application** and go back to the **Discover** menu of Kibana.
    #. Note the IP Address which has been collected by hover over the field **xff_ip**.
    #. You´ll notice the same Device ID A identifier is used from two different IP addresses.

        .. image:: ../pictures/module2/img_class3_module2_animated_10.gif
           :align: center
           :scale: 30%

    #. Within the Kibana **Dashboard** you got the visualized output of your before carried out tasks.
    #. So access to **Arcadia Application** has been made from two different Source IPs, however, there is only one **Device ID Type A** / **Device ID Type B** identifier generated.

        .. image:: ../pictures/module2/img_class3_module2_animated_11.gif
           :align: center
           :scale: 30%


**Demo some Use Cases - Unusual Devices accessing user accounts**

    #. Within that demo we will try to login to **Arcadia Application** with different username as well we change the Source IP of the request.
    #. You either do it manually by using the Browser or use Postman.
    #. In this example I´ll use Postman. 
    #. Open Postman from your Desktop and open the collection called "Device ID+ ELK". Here you´ll find one request.
    #. Within the request you could modify either the Body called **username** or the Header called **x-forwarded-for**.

        .. image:: ../pictures/module2/img_class3_module2_animated_12.gif
           :align: center
           :scale: 30%

    #. In the example I´ll simulate two login requests using two different username and two different Source IPs. 

        .. image:: ../pictures/module2/img_class3_module2_animated_13.gif
           :align: center
           :scale: 30%

    #. Within the Dashboard of Kibana you´ll see those two requests generated only one **Device ID Type A** / **Device ID Type B**.

        .. image:: ../pictures/module2/img_class3_module2_static_2.gif
           :align: center
           :scale: 30%

    #. However, those requests came from two different source IPs (two Region).

        .. image:: ../pictures/module2/img_class3_module2_static_3.gif
           :align: center
           :scale: 30%
    
    #. The Ratio between **Device ID Type A** / **Device ID Type B** and **Source IPs** is 1:2 (1 Device ID to 2 Source IPs).

        .. image:: ../pictures/module2/img_class3_module2_static_4.gif
           :align: center
           :scale: 30%

    #. The Ratio between **Device ID Type A** / **Device ID Type B** and **Username** is 1:2 (1 Device ID to 2 Username).

        .. image:: ../pictures/module2/img_class3_module2_static_5.gif
           :align: center
           :scale: 30%

