# Easy Chat

[Easy Chat](https://paul-gauthier.github.io/easy-chat/)
is a user interface for ChatGPT, designed specifically for young readers.
Almost all the code for Easy Chat was written by ChatGPT.

## Features

- Typography, layout and text spacing which is helpful for young, early or struggling readers.
- Large, easy-to-read text in the Open Dyslexic font.
- Text-to-speech of individual words by clicking on any word.
- Text-to-speech of entire passages with Immersive Reading (highlights each word as it is read aloud at a moderate reading speed). Click on the speaker icons.
- A system prompt that tells ChatGPT it is speaking to a child and requesting it to use common words, short sentences and short paragraphs.

![Screenshot of Easy Chat](screenshot.gif)

## Usage

Try it out here: [Easy Chat](https://paul-gauthier.github.io/easy-chat/)

## Created by ChatGPT

Almost all the code in this repository was written by ChatGPT.
I started by asking it to create the html for a simple chat app, with embedded css and js.
After that, I kept feeding it the entire html/js/css blob and requesting changes, bug fixes, new features and improvements.

### Example prompts

Most of my prompts were basically feature requests, like you might file in a JIRA ticket.
Here's an example prompt:

> Use text-to-voice to speak the highlighted word.

Which resulted in
[these changes](https://github.com/paul-gauthier/easy-chat/commit/62e0862ce0cf1017082e30ec7fa4034cfaf80137) to the code:

```
diff --git a/chat.html b/chat.html
index 4ed5668..7d3d563 100644
--- a/chat.html
+++ b/chat.html
@@ -206,8 +206,17 @@
                         const highlightedWords = chatBox.querySelectorAll('.highlight');
                         highlightedWords.forEach(word => word.classList.remove('highlight'));
                         event.target.classList.add('highlight');
+                        const textToSpeak = event.target.textContent;
+                        const speech = new SpeechSynthesisUtterance(textToSpeak);
+                        speechSynthesis.speak(speech);
                     }
                 });
+
+                /* New code for text-to-speech */
+                const speak = (text) => {
+                    const speech = new SpeechSynthesisUtterance(text);
+                    speechSynthesis.speak(speech);
+                };
        </script>
 </body>
 </html>
```                        

Here is another prompt, this time describing a bug and asking for a bug fix:

> The text-to-speech is saying "speaker outputting high volume" at the end of every message.
> I think it is reading the unicode speaker icon aloud.
> It should not.
> Fix this bug.

Which produced
[this simple fix](https://github.com/paul-gauthier/easy-chat/commit/2e73c58dccc4336f53264dd6b9b5093cf88b0d20):

```
diff --git a/chat.html b/chat.html
index b9f8ba3..8f6de60 100644
--- a/chat.html
+++ b/chat.html
@@ -207,7 +207,7 @@

                 /* New code for word highlighting */
                 chatBox.addEventListener('click', (event) => {
-                    if (event.target.tagName === 'SPAN') {
+                    if (event.target.tagName === 'SPAN' && !event.target.classList.contains('speaker')) {
                         const highlightedWords = chatBox.querySelectorAll('.highlight');
                         highlightedWords.forEach(word => word.classList.remove('highlight'));
                         event.target.classList.add('highlight');
```

One of the most impressive changes was when I asked it to take what was currently a non-functional chat UI and
[wire it up to the OpenAI chat completions API](https://github.com/paul-gauthier/easy-chat/commit/61326c036fa7888e58231f4bcb4f13d0f889ea0c).
I don't have a record of the exact prompt I used, but I basically said "wire it up like this" and pasted
a dozen lines of the `curl` example from the [API reference docs](https://platform.openai.com/docs/api-reference/chat).

I didn't start recording the actual prompts I was using until I was a couple of dozen commits into the process.
I was kind of suprised at how much I was accomplishing by treating ChatGPT as a junior web developer.

Regardless, you can see many of the prompts I used in the
[git history of this repo](https://github.com/paul-gauthier/easy-chat/commits/main).
Any commit that starts with "PROMPT" was created by ChatGPT.
If the commit starts with "asked for ...", that also means ChatGPT did the coding, but I didn't record the exact prompt.

### Workflow

My workflow for each change was dead simple:

  - I wrote my plain English change request in a file called `prompt.txt`.
  
  - I fed the prompt and the entire codebase to ChatGPT via the [aichat](https://github.com/sigoden/aichat) tool.
    - `cat prompt.txt chat.html | aichat -r webdev > tmp.html && cp tmp.html chat.html`

  - Each time, ChatGPT returned a modified version of the codebase, implementing my requested changes.
  
  - I did a `git diff` to review what ChatGPT had changed and tried out the resulting system.
    - If it was good, `git commit`
    - If it wasn't right, I would use `git stash` to discard the changes, adjust the prompt and try again.

I used the roles feature of [aichat](https://github.com/sigoden/aichat) to set up a `webdev` role with a system prompt like this:

> I want you to act as a web development expert.
> I want you to answer only with code.
> Make the requested change to the provided code and output the changed code.
> MAKE NO OTHER CHANGES!
> Do not provide explanations!

### Limitations

My workflow broke down when the size of the prompt and codebase exceeded the context window for ChatGPT 3.5 turbo.
I experimented with numerous other workflows to try and work around this limit, but with little consistent success.

Ultimately I refactored the js and css into their own files, and began feeding ChatGPT excerpts from the code that were relevant to each change I needed.
This wasn't always successful, as it often wanted to (re)write the code that I wasn't showing it.

I look forward to GPT-4 API access, where I can take advantage of the 32k token context window.

