---
author: Patrick O'Connor
paging: Slide %d / %d
---

```
~~~figlet -c -t -f ANSI_Shadow "FZF: FUZZY FINDING YOUR TERMINAL"

~~~

```

```
~~~display_center
[link 2022]

~~~
```
---
# EFF-ZEE-WHAT?
- Fuzzy searcher for the terminal
- Give it some input and start typing to narrow down your search
---
# Baby's first fuzzy search
```bash
printf '1\n2\n3\n' | fzf-tmux -h 20
```

---
# Okay maybe something more useful?
>This will find the top level files in my repo and then allow me to select one of
>them. The selected file will be sent to stdout.

```bash
ls ~/workspace/packager-service | fzf-tmux -h 20
```
---

# Okay maybe something more useful?
>This will find the top level files in my repo and then allow me to select one of
>them. The selected file will be sent to stdout.

```bash
ls ~/workspace/packager-service | fzf-tmux -h 20
```

So we have a way to fuzzy find a file and then send that to stdout.
But the input to fzf could be anything. It doesn't need to be a file at all.
---
# Something even more useful
> Here's a small snippet that let's you list the current tabs and panes in tmux.
> I'll feed this into fzf and when I select the one I want it will go to that
pane. This has been really useful when working in multiple repos at the same time.

```bash
selected_tuple=($(tmux list-panes -s -F '#{window_index} #{pane_index} #{pane_current_path} #{pane_current_command}' | fzf-tmux -h 20))
tmux select-window -t "${selected_tuple[0]}"
tmux select-pane -t "${selected_tuple[1]}"
```
---
# "Pat, I'm not a baby. Show me the complicated stuff" -- You
>Alright, I got one for you.

> Here's a script to use Github's API to find PR's that request your review. It
shows the name of the PR. When you select it, it opens the PR in the browser for
you to review. Is that complicated enough for you?

```bash

# Get the time in seconds 10 days ago
since=$(date -Iseconds -d '10 days ago')
export GH_TOKEN='<YOUR TOKEN WITH NOTIFICATION AND REPO ACCESS>'
export GH_USER='<YOUR USERNAME>'

# Get 10 "notifications" since 10 days ago in json
json=$(curl --silent -u "${GH_USER}":"${GH_TOKEN}" "https://api.github.com/notifications?per_page=10&since=${since}")

# Create tuples from the json using jq to filter for "review_requested". The
# output will look like a list of json objects with title and url.
pairs=$(echo ${json} | jq '.[] | select(.reason == "review_requested") | [{title: .subject.title, url: .subject.url}]')

# Pass the titles into fzf and wait for a selection
selected=$(echo "${pairs}" | jq '.[].title' | fzf-tmux -h 20)
[[ "$selected" = "" ]] && exit

# Re-parse the json and look for the url that matches the title that was
# selected.
selected_url=$(echo ${json} | jq '.[] | select(.subject.title == '"$selected"') | .subject.url' | sed 's/"//g')

# We need to use curl again to get the html url. Then we just use xdg-open (or
# open on Mac) to open the url in our browser. EASY.
curl --silent -u "${GH_USER}":"${GH_TOKEN}" "${selected_url}" | jq '.html_url' | xargs -i{} nohup xdg-open {} > /dev/null 2>&1 &


```
---
# Thanks for coming to my talk. Here's some links
- [Repo with these slides](https://github.com/PatOConnor43/link2022fzf)
- [fzf](https://github.com/junegunn/fzf)
- [Terminal slide application](https://github.com/maaslalani/slides)
- [Figlet fonts](https://github.com/xero/figlet-fonts)

