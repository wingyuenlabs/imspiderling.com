---
Title: Youtube Downloader - My first MCP Server
Description: MCP is a protocol designed to simplify how applications provide context to LLMs and AI agents. Think of it as the USB-C port of AI applications.
Author: 
Date: 2025-07-25
Robots: noindex,nofollow
Template: index
tagline: Youtube Downloader - My first MCP Server
---
Hi everyone, recently, after reading some blogs and content about MCP on the internet, I decided to experiment with it by creating a local MCP server to perform a simple task: downloading a YouTube video. This post is to talk a bit about MCP and the solution I built.

You might ask, why download YouTube videos? Well, during my recent trip from the Netherlands to Brazil, which takes around 12 hours, I wanted to watch some tech and fun videos offline. I know YouTube offers a subscription that allows offline access, but I wasn’t planning to pay for another subscription just for this one trip.

So, instead of using random online tools or subscribing to YouTube Premium, I decided to create an MCP server to help me with the task.

But before I talk about the solution, let me give you a quick overview of MCP.

## Model Context Protocol (MCP)

MCP is a protocol designed to simplify how applications provide context to LLMs and AI agents. Think of it as the USB-C port of AI applications.

MCP follows a client-server architecture, with the Host, Client, and Server concepts.

The MCP Host is an AI application like Claude Code or Claude Desktop, which establishes connections to one or more MCP servers. The Host does this by creating one MCP Client for each server. Each Client maintains a dedicated one-to-one connection with its corresponding MCP Server. The Server is responsible for providing features and functionalities to the host.

There are two different types of communication channels (called Transport Layers) between the Client and Server. One of them is Stdio transport, which is used for local execution. It communicates via standard input/output streams between processes on the same machine.

The other one is the Streamable HTTP transport, which uses HTTP POST for client-to-server messages with optional Server-Sent Events for streaming capabilities. This transport enables remote server communication and supports standard HTTP authentication methods including bearer tokens, API keys, and custom headers.

In our current solution, we’re using the Stdio transport.

## The Solution

The solution was written in Python and uses the FastMCP library to handle MCP server creation. With this library, you can create MCP tools, which are defined as Python functions that return responses as strings.

I defined two endpoints (MCP tools). One to download the YouTube video, running the process in the background. And another to check the status of the download, since it runs in the background, we want to be able to track its progress. You can find the endpoint definitions inside `server/endpoints.py`.

The YouTube download logic is defined in `server/services.py`. We’re using the `yt_dlp` library to handle the download process and communication. This library uses `FFmpeg` to merge the audio and video files into a single .mp4 output.

You can check more details and follow the instructions to run this project by visiting the repository. If you liked it, please give it a ⭐️ on GitHub! If you want to make improvements, feel free to fork and open a PR. And if you have any questions, drop them in the comments section below.

[https://github.com/SamuelBFavarin/youtube-downloader-mcp-server](https://github.com/SamuelBFavarin/youtube-downloader-mcp-server)

Check out the screenshots to see the MCP Server in action. Thank you, and see you soon!

[![Download Prompt](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd5r978n2kq07w5nu1zlv.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd5r978n2kq07w5nu1zlv.png)

[![Checking Status](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxyragko8hmw6y3z99ba2.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxyragko8hmw6y3z99ba2.png)

[![Checking Status again](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frbtkeuurzp88da3rj16b.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frbtkeuurzp88da3rj16b.png)

