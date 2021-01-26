Module 2: Check how Device ID+ works
####################################

In this module, we will work with Device ID+ and get the identifier reported back into /var/ltm/log of BIG-IP.

.. warning:: This is not a full integration to show a Device ID+ / AWF Demo. Currently we are working on Use Cases which can be integrated into the Demo.

.. note:: We are currently working on a use case to leverage Device ID+ together with AWF and will update the Module accordingly. If you got a specific AWF "Device ID+ Usecase" in mind, please reach out to Patrick Zoller.

**Device ID+ Overview**

DeviceID+ is a pseudonymized identifier that can be used to identify a device visiting a customer's applications. 
More precisely, it is an identifier for a browser or a native mobile application. For the initial release of DeviceID+, we are only focusing on Web and Mobile Web applications.

.. note:: If you haven´t worked with Device ID+ before, please review the `Device ID+`_ Article on F5 Cloud Services.

.. _`Device ID+` : https://clouddocs.f5.com/cloud-services/latest/f5-cloud-services-DeviceID-About.html


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

.. _`Getting Started with F5 Device ID+` : https://clouddocs.f5.com/cloud-services/latest/f5-cloud-services-DeviceID-GettingStarted.html#getting-started-with-f5-device-id


**Device ID+ and iRule**

Device ID+ includes two identifiers – a residue-based identifier and an attribute-based identifier. The residue-based identifier is based on local storage and cookies. 
The attribute-based identifier is based on signals collected on the device. The two identifiers always have different values.

1JS writes both the residue-based and attribute-based identifiers in a single, first-party cookie called *_imp_apg_r_*. The *_imp_apg_r_* cookie is URL encoded with the following format:

%7B%22diA%22%3A%22AT9cyV8AAAAAd60uXCtYafPTZGLaVAku%22%2C%22diB%22%3A%22ASJ4gFmzPo%2Fa8AHJceWhykudRoXeBGlP%22%7D

This cookie can be decoded via `URL Decoder`_ to get the response in clear text. The decoded cookie has the following format:
.. _`URL Decoder` : https://www.urldecoder.org/

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