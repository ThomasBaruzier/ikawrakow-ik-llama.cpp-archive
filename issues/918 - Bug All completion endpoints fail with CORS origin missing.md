## 📌 [Issue #918](https://github.com/ikawrakow/ik_llama.cpp/issues/918) - Bug: All completion endpoints fail with CORS origin missing

| **Author** | `nauful` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-11-07 |
| **Updated** | 2025-11-09 |

---

## 📄 Description

### What happened?

The latest build fails CORS checks for all completions endpoints:

<img width="3212" height="1520" alt="Image" src="https://github.com/user-attachments/assets/37323f0c-b902-4163-87c9-d9e0b5514854" />

The built-in webui works as the origin is exactly the same, but requests from other ports or hosts fail. I don't know if this is a good way to fix the issue, but I made these changes as a local fix.
```diff
 examples/server/server.cpp | 4 ++++
 1 file changed, 4 insertions(+)

diff --git a/examples/server/server.cpp b/examples/server/server.cpp
index 4428af24..5d9c2de5 100644
--- a/examples/server/server.cpp
+++ b/examples/server/server.cpp
@@ -4413,6 +4413,7 @@ int main(int argc, char ** argv) {
     const auto handle_completions = [&handle_completions_impl](const httplib::Request & req, httplib::Response & res) {
         auto data = json::parse(req.body);
         std::vector<raw_buffer> files; // dummy
+        res.set_header("Access-Control-Allow-Origin", req.get_header_value("Origin"));
         handle_completions_impl(
             SERVER_TASK_TYPE_COMPLETION,
             data,
@@ -4425,6 +4426,7 @@ int main(int argc, char ** argv) {
         auto body = json::parse(req.body);
         json data = oaicompat_chat_params_parse(body);
         std::vector<raw_buffer> files; // dummy
+        res.set_header("Access-Control-Allow-Origin", req.get_header_value("Origin"));
         handle_completions_impl(
             SERVER_TASK_TYPE_COMPLETION,
             data,
@@ -4458,6 +4460,7 @@ int main(int argc, char ** argv) {
         auto body = json::parse(req.body);
         std::vector<raw_buffer> files;
         json data = oaicompat_chat_params_parse(ctx_server.model, body, ctx_server.oai_parser_opt, files);
+        res.set_header("Access-Control-Allow-Origin", req.get_header_value("Origin"));
         handle_completions_impl(
             SERVER_TASK_TYPE_COMPLETION,
             data,
@@ -4482,6 +4485,7 @@ int main(int argc, char ** argv) {
         ctx_server.queue_results.add_waiting_task_id(id_task);
         ctx_server.request_completion(id_task, -1, data, true, false, std::move(token));
         std::vector<raw_buffer> files; // dummy
+        res.set_header("Access-Control-Allow-Origin", req.get_header_value("Origin"));
         handle_completions_impl(
             SERVER_TASK_TYPE_INFILL,
             data,
```

### Name and Version

version: 3962 (6a805c73)
built with MSVC 19.44.35219.0 for x64

### What operating system are you seeing the problem on?

Windows

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-11-07** at **18:54:52**

Thanks. Will fix.

---

👤 **firecoperana** commented on **2025-11-08** at **20:51:48**

Can you check if https://github.com/ikawrakow/ik_llama.cpp/pull/924 fixed it?

---

👤 **nauful** commented on **2025-11-09** at **00:49:18**

It did not. The completions endpoints still don't return Access-Control-Allow-Origin.

---

👤 **firecoperana** commented on **2025-11-09** at **01:10:57**

I forgot one line. Check again. .

---

👤 **nauful** commented on **2025-11-09** at **12:28:11**

Yes thanks, that fixed it.