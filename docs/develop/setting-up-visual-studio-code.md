---
layout: page
parent: Develop ACAP applications
title: Setting up Visual Studio Code
nav_order: 8
---

# Setting up Visual Studio Code

Visual Studio Code provides access to containerized development tools, without the need for you to install them natively on your development computer.

To start developing ACAP applications using Visual Studio Code:

1. Install the **[Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)** extension available in the **Extensions Marketplace** in **Visual Studio Code**.
2. Create a subfolder called `.devcontainer` in the top directory of the source code project you are working on.
In `.devcontainer`, create a `devcontainer.json` with the following content:

   ```json
   {
      "name": "My acap-native-sdk container",
      "image": "axisecp/acap-native-sdk:1.8-<ARCH>-ubuntu22.04"
   }
   ```

   `<ARCH>` being either `armv7hf` or `aarch64`

3. Press **Ctrl+Shift+P** and type in `Dev Containers: Open Folder in Container`.

The application restarts and is now attached to a container with the SDK and your code. This way, you can interactively edit your source code just as if the tools had been installed natively, including using all the debugging and support features in Visual Studio Code.

You can install different versions of the SDK in separate containers. When you open your source code folder, Visual Studio Code identifies the SDK version defined in the `devcontainer.json`.

The ACAP Native SDK container includes all the SDK tools, Git and some other useful things. But you can create your own Dev Container Dockerfile or add more tools to the `devcontainer.json` configuration. See [Microsoft's tutorials on how to use Development Containers](https://code.visualstudio.com/docs/remote/containers) for more information on what this way of working can offer.

## Code completion of Axis APIs

To access the header files of the Axis APIs inside the Development container, the following architecture independent path should be added to the VS Code extension configuration of your choice:

```text
${SDKTARGETSYSROOT}/usr/include/
```

### Microsoft C/C++ extension

To enhance your development experience in Visual Studio Code, you can install the Microsoft [C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools). This extension provides various features that can assist you with code editing and navigation.

To install and setup the extension, follow these steps:

1. Install the [C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)  available in the **Extensions Marketplace** in **Visual Studio Code**.
2. Create a subfolder called `.vscode` in the top directory of the source code project you are working on.
In `.vscode`, create a `c_cpp_properties.json` with the following content:

   ```json
   {
       "configurations": [
           {
               "name": "Linux",
               "includePath": [
                   "${workspaceFolder}/**",
                   "${SDKTARGETSYSROOT}/usr/include/**"
               ]
           }
       ],
       "version": 4
   }
   ```

Once the installation is complete, you can benefit from features such as IntelliSense and code browsing.
You can now, for example get suggestions about Axis libraries as you type or hover over a symbol:

<!-- markdownlint-disable MD033 -->
<img src="../../assets/images/vs-code-ms-cpp-extension.png" height="240">

For detailed information on how to use the C/C++ extension and its various features, refer to the extension's documentation available in the [Visual Studio Code Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools).
