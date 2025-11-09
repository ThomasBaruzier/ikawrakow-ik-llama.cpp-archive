## 🔀 [Pull Request #653](https://github.com/ikawrakow/ik_llama.cpp/pull/653) - Add GitHub data: backup and convertion scripts + backup update

| **Author** | `ThomasBaruzier` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Source Branch** | `tb/github-data-scripts` |
| **Target Branch** | `main` |
| **Created** | 2025-07-26 |
| **Updated** | 2025-09-12 |

---

## 📄 Description

Hello!  

I've refined the scraping and conversion scripts. While they should work with any repository, I haven't extensively tested them beyond the current use case. For this repository, the scripts consistently complete in ~30 seconds (initially 750s!) using just ~~11~~ 10 API requests to fetch all issues, pull requests, and discussions.  

I initially explored resumable/incremental scraping but abandoned the idea due to reliability issues: the `updatedAt` field only reflects edits to the issue/PR body, not new activity. Instead, I focused on optimization, achieving the results below.  

---

### Usage  
```bash
export GITHUB_TOKEN='github_pat_...'
cd github-data
rm -rf issues discussions pull_requests index.md ik.json
python ghscrape.py ikawrakow/ik_llama.cpp -o ik.json
python ghconvert.py ik.json -o .
```

#### Or as a one-liner (ensure that you are in `github-data/`):
```bash
rm -rf issues discussions pull_requests index.md ik.json && GITHUB_TOKEN='github_pat_...' python ghscrape.py ikawrakow/ik_llama.cpp -o ik.json && python ghconvert.py ik.json -o .
```

---

### Scraping Demo

```
python ghscrape.py ikawrakow/ik_llama.cpp -o ik.json
```
```
I: Fetching all issues...
I: API Rate Limit (Req #1): 4997 points remaining, resets in 59m 54s.
I: Processed 100 issues...
I: API Rate Limit (Req #2): 4994 points remaining, resets in 59m 52s.
I: Processed 131 issues...
I: Fetching all nested data for 131 items (1 pages)...
I: API Rate Limit (Req #3): 4993 points remaining, resets in 59m 52s.
I: Processed batch of 1 pages. 0 pages remaining.
I: Structuring final items for issues...
I: Finished issues: Found and processed 131 items.
I: Fetching all pull requests...
I: API Rate Limit (Req #4): 4888 points remaining, resets in 59m 49s.
I: Processed 100 pull_requests...
I: API Rate Limit (Req #5): 4783 points remaining, resets in 59m 46s.
I: Processed 200 pull_requests...
I: API Rate Limit (Req #6): 4678 points remaining, resets in 59m 41s.
I: Processed 300 pull_requests...
I: API Rate Limit (Req #7): 4573 points remaining, resets in 59m 36s.
I: Processed 400 pull_requests...
I: API Rate Limit (Req #8): 4468 points remaining, resets in 59m 34s.
I: Processed 452 pull_requests...
I: Fetching all nested data for 452 items (0 pages)...
I: Structuring final items for pull_requests...
I: Finished pull_requests: Found and processed 452 items.
I: Fetching all discussions...
I: API Rate Limit (Req #9): 4366 points remaining, resets in 59m 30s.
I: Processed 71 discussions...
I: Fetching all nested data for 71 items (1 pages)...
I: API Rate Limit (Req #10): 4365 points remaining, resets in 59m 29s.
I: Processed batch of 1 pages. 0 pages remaining.
I: Structuring final items for discussions...
I: Finished discussions: Found and processed 71 items.
I: Data successfully saved to ik.json
I: Total execution time: 28.77 seconds
```

### Conversion Demo

```
python ghconvert.py ik.json -o .
```
```
Processing 131 issues...
Processing 452 pull_requests...
Processing 71 discussions...
Generating index.md summary file...
Successfully generated 654 Markdown files.
Files are in the '.' directory.
```

---

### Relevant links jump:

Scripts:
- https://github.com/ThomasBaruzier/ik_llama.cpp/blob/tb/github-data-scripts/github-data/ghscrape.py
- https://github.com/ThomasBaruzier/ik_llama.cpp/blob/tb/github-data-scripts/github-data/ghconvert.py

Index:
- https://github.com/ThomasBaruzier/ik_llama.cpp/blob/tb/github-data-scripts/github-data/index.md

Discussion example:
- https://github.com/ThomasBaruzier/ik_llama.cpp/blob/tb/github-data-scripts/github-data/discussions/477%20-%20DeepSeek-R1-0528%20ik%20quants.md

PR example:
- https://github.com/ThomasBaruzier/ik_llama.cpp/blob/tb/github-data-scripts/github-data/pull_requests/620%20-%20Bump%20Windows%20max%20open%20files%20from%20512%20to%202048.md

Issue example:
- https://github.com/ThomasBaruzier/ik_llama.cpp/blob/tb/github-data-scripts/github-data/issues/296%20-%20Possible%20numerical%20stability%20issue%20with%20experimental%20quant%20of%20DeepSeek-V3-0324.md

---

### Notes  
- ~~Content extraction for reviews isn’t fully implemented yet (see [example](https://github.com/ThomasBaruzier/ik_llama.cpp/blob/tb/github-data-scripts/github-data/pull_requests/620%20-%20Bump%20Windows%20max%20open%20files%20from%20512%20to%202048.md)). This could be added later if needed.~~ Fixed.
- Wiki backups are not implemented.
- Scripts and filenames also work on Windows.

- [x] I’ve read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md).  
- **Self-reported review complexity**:  
  - [ ] Low
  - [x] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-07** at **05:18:00**

Thank you for this!

But while on vacation I started wondering if scrapping GitHub data does not somehow violate their ToS. Anyone knows?

---

👤 **ThomasBaruzier** commented on **2025-08-07** at **06:55:58**

https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies

> You may use information from our Service for the following reasons, regardless of whether the information was scraped, collected through our API, or obtained otherwise:
> - Researchers may use public, non-personal information from the Service for research purposes, only if any publications resulting from that research are open access.
> - Archivists may use public information from the Service for archival purposes.

It seems that it doesn't break ToS 👍 (I probably should have looked up this before working on this, but hey, it's there)

---

👤 **ikawrakow** commented on **2025-08-07** at **08:29:38**

Maybe I'm too paranoid after having been locked out from my GH account, but can I count as a "Researcher" or as an "Archivist"?

Also, even if the actual scrapping does not violate their ToS, does publishing software that enables scrapping perhaps violate their ToS? Someone who is not an archivist or a researcher who publishes the scrapped data under open access can take the code and do something with it that violates their ToS. Similar to the Pirate Bay not actually engaging in copyright violation, but enabling others to easily do so. Or youtube-dl, which IIRC was taken down due to a DMCA violation complaint.

---

👤 **ThomasBaruzier** commented on **2025-08-07** at **09:14:27**

I mean, this is clearly for archival purposes, and you're using your own token with GitHub's official GraphQL API, which has its own rate limiting to prevent overuse. To be more precise, a full backup of the current repository costs 700 points out of the 5,000 available (which reset hourly). Even if this were done weekly, it shouldn’t draw much attention.  

That said, you’re probably right about the second part, I’m not entirely sure if publishing tools like this complies with GitHub’s ToS. I did find a similar project still active here: https://gist.github.com/animusna/b64b45d910dd3df7cd41ee0f99082766

Nevertheless, I’ve published it under my own account here: https://github.com/ThomasBaruzier/ghbkp

If you’d prefer to keep this separate from your project, I could set up daily or weekly backups there instead. But again, it should be safe. I understand your caution after the two-day takedown we experienced, but that was likely a mistake on GitHub’s part, maybe the sudden influx of stars from new users enjoying Kimi triggered their bot detection and forced an automated takedown?

---

👤 **ubergarm** commented on **2025-09-11** at **15:38:38**

@ThomasBaruzier 

Thanks for your repo! I've been having trouble with github search in general. For fun I tried to "vibe code" a quick github repo CLI tool with the latest Kimi-K2-Instruct, but it was very inefficient making a bunch of requests. Your solution with graphql is quite efficient and much faster.

Now I 

Here are quick instructions for anyone who wants to try it which takes just a few minutes:

```bash
# install requests is only python dependency
$ uv venv ./venv --python 3.12 --python-preference=only-managed
$ source ./venv/bin/activate
$ uv pip install requests

# get the scripts
$ git clone git@github.com:ThomasBaruzier/ghbkp.git

# Add fine-grained token with read only access with 30 day expiration etc...
# https://github.com/settings/personal-access-tokens

# save data for easy searching locally
$ source ./venv/bin/activate
$ export GITHUB_TOKEN=COPYPASTETOKENJUSTMADEABOVE
$ python ghbkp/ghscrape.py ikawrakow/ik_llama.cpp

I: Fetching all issues...
I: API Rate Limit (Req #1): 4997 points remaining, resets in 59m 52s.
I: Processed 100 issues...
I: API Rate Limit (Req #2): 4994 points remaining, resets in 59m 48s.
I: Processed 174 issues...
I: Fetching all nested data for 174 items (1 pages)...
I: API Rate Limit (Req #3): 4993 points remaining, resets in 59m 48s.
I: Processed batch of 1 pages. 0 pages remaining.
I: Structuring final items for issues...
I: Finished issues: Found and processed 174 items.
I: Fetching all pull requests...
I: API Rate Limit (Req #4): 4888 points remaining, resets in 59m 45s.
I: Processed 100 pull_requests...
I: API Rate Limit (Req #5): 4783 points remaining, resets in 59m 41s.
I: Processed 200 pull_requests...
I: API Rate Limit (Req #6): 4678 points remaining, resets in 59m 35s.
I: Processed 300 pull_requests...
I: API Rate Limit (Req #7): 4573 points remaining, resets in 59m 28s.
I: Processed 400 pull_requests...
I: API Rate Limit (Req #8): 4468 points remaining, resets in 59m 20s.
I: Processed 500 pull_requests...
I: API Rate Limit (Req #9): 4363 points remaining, resets in 59m 18s.
I: Processed 514 pull_requests...
I: Fetching all nested data for 514 items (1 pages)...
I: API Rate Limit (Req #10): 4362 points remaining, resets in 59m 18s.
I: Processed batch of 1 pages. 1 pages remaining.
I: API Rate Limit (Req #11): 4361 points remaining, resets in 59m 18s.
I: Processed batch of 1 pages. 0 pages remaining.
I: Structuring final items for pull_requests...
I: Finished pull_requests: Found and processed 514 items.
I: Fetching all discussions...
I: API Rate Limit (Req #12): 4259 points remaining, resets in 59m 12s.
I: Processed 84 discussions...
I: Fetching all nested data for 84 items (1 pages)...
I: API Rate Limit (Req #13): 4258 points remaining, resets in 59m 12s.
I: Processed batch of 1 pages. 0 pages remaining.
I: Structuring final items for discussions...
I: Finished discussions: Found and processed 84 items.
I: Data successfully saved to ikawrakow__ik_llama.cpp.json
I: Total execution time: 48.67 seconds

# check the data
$ cat ikawrakow__ik_llama.cpp.json | jq '.discussions[].author' | sort | uniq -c | sort -n | tail -n 5
      2 "usrlocalben"
      5 "magikRUKKOLA"
      6 "ubergarm"
      7 "Nexesenex"
     19 "ikawrakow"
```

Now I can easily `grep` to find things!

---

👤 **ThomasBaruzier** commented on **2025-09-11** at **23:51:08**

Thanks for the demo, I'm glad you found it useful!

Have you tried ghconvert.py? I may add built-in filtering if relevant.

---

👤 **ubergarm** commented on **2025-09-12** at **15:20:03**

Oh no I didn't realize what that does! I'll try it:

Ok, Looks like your `ghconvert.py` will take the `.json` file produced by `ghscrape.py` and create a directory structure filled with an index markdown file and discussions/issues/pull_requests subdirs with comments inserted into each one.

```bash
$ python ghbkp/ghconvert.py ikawrakow__ik_llama.cpp.json -o ./outputdir/
$ tree -L 1 ./outputdir/
.
├── discussions
├── index.md
├── issues
└── pull_requests
```

This is great because I can `grep -ri foobar` then get a quick reference link from the top of the file to share references with folks! 

Thanks for this useful tool!