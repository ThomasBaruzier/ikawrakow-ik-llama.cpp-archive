## 🗣️ [Discussion #770](https://github.com/ikawrakow/ik_llama.cpp/discussions/770) - How can I access the ik-llama.cpp server from another computer?

| **Author** | `edyapd` |
| :--- | :--- |
| **State** | ✅ **Answered** |
| **Created** | 2025-09-09 |
| **Updated** | 2025-09-17 |

---

## 📄 Description

I'm trying to access the ik-llama.cpp server from another computer on my local network, but I'm running into an issue.

Setup:
The server is running on a machine with the IP address 192.168.0.110.
I started the server with the --host 0.0.0.0 flag to allow external connections.
The client machine is on the same network with the IP address 192.168.0.101.

Behavior:
When I access http://192.168.0.110:8080/ from a web browser on the same machine (the server machine), the web UI loads correctly. The server log shows a successful request:

```
INFO [      log_server_request] request | tid="5832" timestamp=1757425277 remote_addr="192.168.0.110" remote_port=59900 status=200 method="GET" path="/" params={}
INFO [      log_server_request] request | tid="5832" timestamp=1757425277 remote_addr="192.168.0.110" remote_port=59900 status=404 method="GET" path="/favicon.ico" params={}
```

However, when I try to access http://192.168.0.110:8080/ from the other computer, the browser receives a 404 error with this JSON response:

```
{"error":{"code":404,"message":"File Not Found","type":"not_found_error"}}
```

The server log for this failed remote request looks like this:

```
INFO [      log_server_request] request | tid="11080" timestamp=1757425238 remote_addr="192.168.0.101" remote_port=61851 status=404 method="GET" path="http://192.168.0.110:8080/" params={}
INFO [      log_server_request] request | tid="11080" timestamp=1757425239 remote_addr="192.168.0.101" remote_port=61851 status=404 method="GET" path="http://192.168.0.110:8080/favicon.ico" params={}

```

Could you please advise on how to properly configure the server for access from other computers on the network? Is this a known issue?

Thank you!

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-09-09** at **15:45:49**

I typically use --port 5000 for the server, and then 192.168.0.110:5000 to connect the client

> 👤 **edyapd** replied on **2025-09-09** at **16:05:29**
> 
> Thank you for your reply.
> But unfortunately it doesn't work for me.
> Client: Windows 7, Chrome.
> I make a request to http://192.168.0.110:5000
> In response I get:
> ```
> {"error":{"code":404,"message":"File Not Found","type":"not_found_error"}}
> ```
> Server: Windows 10
> Message in console:
> ```
> INFO [ main] HTTP server listening | tid="10940" timestamp=1757433452 hostname="0.0.0.0" port="5000" n_threads_http="31"
> INFO [ update_slots] all slots are idle | tid="10940" timestamp=1757433452
> INFO [ log_server_request] request | tid="11152" timestamp=1757433469 remote_addr="192.168.0.101" remote_port=63814 status=404 method="GET" path="http://192.168.0.110:5000/" params={}
> INFO [ log_server_request] request | tid="11152" timestamp=1757433469 remote_addr="192.168.0.101" remote_port=63814 status=404 method="GET" path="http://192.168.0.110:5000/favicon.ico" params={}
> ```

> 👤 **ikawrakow** replied on **2025-09-09** at **16:27:03**
> 
> Remove --host 0.0.0.0 and add --port 5000 to your server command

> 👤 **edyapd** replied on **2025-09-09** at **17:18:49**
> 
> The server runs on a local address by default. I can't access it from another computer.
> ```
> INFO [ main] HTTP server listening | tid="5128" timestamp=1757437264 hostname="127.0.0.1" port="5000" n_threads_http="31"
> INFO [ update_slots] all slots are idle | tid="5128" timestamp=1757437264
> ```

---

👤 **mcm007** commented on **2025-09-09** at **16:26:21**

Client IP appears on the server logs, it means that firewall is not blocking the port.

You could try one of the API endpoints with `curl` to rule out browser issues, example:

`curl -k http://192.168.0.110:8080/v1/chat/completions -H "Content-Type: application/json" -d '{
    "model": "Qwen_Qwen3-30B-A3B-IQ4_KS.gguf",
    "messages": [
    {
        "role": "user",
        "content": "Hello!"
    }
    ]
    }'`

> 👤 **edyapd** replied on **2025-09-09** at **17:23:45**
> 
> Thanks for trying to help.
> I don't have the curl command in Windows 7.
> ChatGPT advised me to send it via the browser console:
> ```
> fetch('http://192.168.0.110:5000/v1/chat/completions', {
>   method: 'POST',
>   headers: {
>     'Content-Type': 'application/json'
>   },
>   body: JSON.stringify({
>     model: 'deepseek-ai_DeepSeek-V3.1-IQ2_S-00001-of-00005.gguf.gguf',
>     messages: [
>       { role: 'user', content: 'Hello!' }
>     ]
>   })
> })
> .then(response => response.json())
> .then(data => console.log(data))
> .catch(error => console.error('Error:', error));
> ```
> This is what I got in response:
> ```
>  POST http://192.168.0.110:5000/v1/chat/completions 404 (Not Found)
>  
> {code: 404, message: 'File Not Found', type: 'not_found_error'}
> ```
> And this is in the server console:
> ```
> INFO [      log_server_request] request | tid="8300" timestamp=1757437884 remote_addr="192.168.0.101" remote_port=64968 status=404 method="GET" path="http://192.168.0.110:5000/" params={}
> INFO [      log_server_request] request | tid="4728" timestamp=1757437938 remote_addr="192.168.0.101" remote_port=65130 status=404 method="POST" path="http://192.168.0.110:5000/v1/chat/completions" params={}
> ```

---

👤 **mcm007** commented on **2025-09-09** at **17:49:08**

Hopefully is not IE :smile: 

Try with recent version of Firefox `http://192.168.0.110:5000/v1/models`, it should return a JSON in browser.

---

👤 **edyapd** commented on **2025-09-10** at **08:37:44**

Thanks to everyone who responded. I figured out what the problem was. I’d spent two days asking ChatGPT about it and it couldn’t answer; today it got it right on the first try.

The issue was that the computer I was sending the request from had a proxy enabled. It was sending the full URL—instead of “/”, the server was receiving “http://192.168.0.110:8080/”, couldn’t handle it, and returned 404.

Sorry for taking up your time.