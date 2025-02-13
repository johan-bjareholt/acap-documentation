---
layout: page
parent: Develop ACAP applications
title: VAPIX access for ACAP applications
nav_order: 7
---

# VAPIX access for ACAP applications - BETA

From AXIS OS 11.6, ACAP applications can acquire VAPIX service account credentials in runtime. With these credentials, the ACAP application can call a local virtual host to make VAPIX requests on the device. A username and a password with high complexity are created for every credential acquisition. These credentials are only valid on the local virtual host (127.0.0.12) and aren't stored in any file. It should only be kept in memory by the ACAP application.

The following steps show how to get VAPIX credentials in an ACAP application.

## Modify the manifest file

- Add the required D-Bus method `com.axis.HTTPConf1.Auth1.GetVapixServiceAccountCredentials` in the manifest file under field `resources.dbus.requiredMethods`.

- Use a dynamic user by removing the `user` section from the manifest. To use this D-Bus service, the ACAP must run as a generated user and it is not available for the general SDK user.

Example Manifest

```json
{
    "schemaVersion": "1.3.1",
    "acapPackageConf": {
        "setup": {
            "appName": "vapix_access",
            "vendor": "Axis Communications",
            "embeddedSdkVersion": "3.0",
            "runMode": "never",
            "version": "1.0.0"
        }
    },
    "resources": {
        "dbus": {
            "requiredMethods": [
                "com.axis.HTTPConf1.Auth1.GetVapixServiceAccountCredentials"
            ]
        }
    }
}
```

## Acquire credentials

An ACAP application can acquire VAPIX service account credentials through a D-Bus call. This example uses GDBus, a high-level D-Bus library for working with D-Bus.

1. Establish a D-Bus connection to the system bus:

    This connection allows the application to communicate over the D-Bus system.

    ```c
    GDBusConnection *connection = g_bus_get_sync(G_BUS_TYPE_SYSTEM, NULL, &error);
    ```

2. Create a proxy object for your D-Bus interface:

    This proxy allows users to interact with your D-Bus service or interface.

    ```c
    GDBusProxy *proxy = g_dbus_proxy_new_sync(connection,
                                                G_DBUS_PROXY_FLAGS_NONE,
                                                NULL, 
                                                "com.axis.HTTPConf1",
                                                "/com/axis/HttpConf1/Auth",
                                                "com.axis.HTTPConf1.Auth1",
                                                NULL, 
                                                &error);
    ```

3. Invoke the D-Bus method using the proxy:

    The user can effectively call the method that returns the credentials by doing this.

    ```c
    GVariant *username = g_variant_new("(s)", "testuser");

    GVariant *result = g_dbus_proxy_call_sync(proxy,
                                                "com.axis.HTTPConf1.Auth1.GetVapixServiceAccountCredentials",
                                                username,  
                                                G_DBUS_CALL_FLAGS_NONE,
                                                -1,  
                                                NULL, 
                                                &error);
    ```

    When the `GetVapixServiceAccountCredentials` method is invoked, it generates credentials with the specified username and a random password, which is returned to the ACAP as a string.

## Make a VAPIX call

After obtaining the credentials, it's ready to make the actual VAPIX call. The ACAP application communicates with a local server, which then checks the given credentials (using basic access authentication) and forwards the VAPIX request.

In this example, the [openssl_curl_example](https://github.com/AxisCommunications/acap-native-sdk-examples/tree/main/utility-libraries/openssl_curl_example) has been used as a baseline and the libcurl C API is used to make the VAPIX call.

1. Define a VAPIX endpoint:

    VAPIX calls are essentially HTTP requests and require a URL.

    ```c
    curl_easy_setopt(curl, CURLOPT_URL, "http://127.0.0.12/axis-cgi/applications/list.cgi");
    ```

2. Generate and set credentials:

    The credentials obtained from the D-Bus call are concatenated and then added to the cURL request.

    ```c
    curl_easy_setopt(curl, CURLOPT_USERPWD, credentials);
    ```

3. Set the authentication method and make the VAPIX call:

    Here, basic access authentication is used, and then the actual cURL request is made.

    ```c
    curl_easy_setopt(curl, CURLOPT_HTTPAUTH, CURLAUTH_ANY);

    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_to_file);

    curl_easy_setopt(curl, CURLOPT_WRITEDATA, &file);
    ```

## Test from command-line

SSH into a device and make a D-Bus call using a command-line to get VAPIX service account credentials.

Example call:

```bash
gdbus call --system
            --dest com.axis.HTTPConf1
            --object-path /com/axis/HttpConf1/Auth
            --method com.axis.HTTPConf1.Auth1.GetVapixServiceAccountCredentials
            "testuser"
```

Example response:

```bash
('id:a0-testuser, pass:MH757eGZdsyBuAbhAQ3j2ZtRNg9xchEg',)
```

Make a VAPIX request on local virtual host 127.0.0.12 with the given credentials using cURL.

Example

```bash
curl -s -u '<id:pass>' http://127.0.0.12/axis-cgi/applications/list.cgi
```

Where `id` and `pass` are the credentials obtained from the GDBus call. This cURL command sends an HTTP GET request to the specified VAPIX endpoint with the required authentication.

## Limitations

- A device can hold a maximum of 200 VAPIX service accounts.
- There are no VAPIX service accounts after rebooting a device. Restarting the device removes all the created credentials.
- The credentials should be re-fetched each time the ACAP application starts and should only be kept in memory by the ACAP application, not stored in any file.
- We officially support only the specific D-Bus method calls introduced in this feature. Use of other D-Bus methods outside of this feature is not supported and may result in undefined behavior.
