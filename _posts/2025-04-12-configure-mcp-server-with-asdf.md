---
layout: post
title: "Configure MCP server with asdf for Node.js"
categories: [mcp, asdf, nodejs, configuration]
---

Today I want to share how to configure an [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) server to run Node.js applications using [asdf](https://asdf-vm.com/) as the version manager. This setup is particularly useful when you need to ensure consistent Node.js versions across different environments and want to leverage asdf's powerful version management capabilities.

## The Challenge

When setting up an MCP server to run Node.js applications, you might encounter issues with Node.js version management and environment variables. The key is to ensure that the MCP server can find and use the correct Node.js version managed by asdf.

## The Solution

Here's a working configuration example for an MCP server that runs a Node.js application:

```json
{
    "mcpServers": {
        "your-mcp-server": {
            "command": "node",
            "args": [
                "/Users/foo/projects/mcp-server/build/index.js"
            ],
            "env": {
                "PATH": "/Users/foo/.asdf/shims:/usr/local/bin:/usr/bin:/bin",
                "ASDF_DIR": "/Users/foo/.asdf",
                "ASDF_DATA_DIR": "/Users/foo/.asdf",
                "ASDF_NODEJS_VERSION": "22.14.0"
            }
        }
    }
}
```

Let's break down the important parts of this configuration:

1. **Environment Variables**:
   - `PATH`: Includes the asdf shims directory first, ensuring that asdf-managed executables are found before system ones
   - `ASDF_DIR`: Points to your asdf installation directory
   - `ASDF_DATA_DIR`: Specifies where asdf stores its data
   - `ASDF_NODEJS_VERSION`: Explicitly sets the Node.js version to use

2. **Command Configuration**:
   - `command`: Set to "node" to use the Node.js executable
   - `args`: Contains the path to your Node.js application's entry point

## Why This Works

The configuration works because it:
- Ensures the correct Node.js version is used by setting the `ASDF_NODEJS_VERSION` environment variable
- Makes asdf-managed executables available by adding the shims directory to the PATH
- Provides all necessary asdf environment variables for proper version management

## Tips for Implementation

1. Make sure you have the Node.js version installed in asdf:
   ```bash
   asdf install nodejs 22.14.0
   ```

2. Verify the version is available:
   ```bash
   asdf list nodejs
   ```

3. Adjust the paths in the configuration to match your system's setup:
   - Replace `/Users/foo` with your home directory
   - Update the application path to point to your Node.js application

4. If you're using a different Node.js version, update the `ASDF_NODEJS_VERSION` accordingly

## Troubleshooting

If you encounter issues:
1. Verify that the Node.js version specified in `ASDF_NODEJS_VERSION` is installed
2. Check that all paths in the configuration are correct
3. Ensure the asdf shims directory is properly set up
4. Try running the Node.js application directly from the command line to verify it works

This configuration provides a robust way to run Node.js applications through MCP while maintaining version consistency through asdf. It's particularly useful in development environments where you need to switch between different Node.js versions for different projects.

For more information about MCP, check out the [official documentation](https://modelcontextprotocol.io/). For asdf, visit the [asdf website](https://asdf-vm.com/) or their [GitHub repository](https://github.com/asdf-vm/asdf).

Happy coding! 🚀