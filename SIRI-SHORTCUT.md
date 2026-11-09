# Talk to Gideon without opening the app

**Why this is needed:** iPhone doesn't let *any* website listen to the microphone in the background or while the screen is locked. Apple blocks that for privacy. So the Gideon web app can only hear you while it's open on screen.

**The way around it:** build a **Shortcut** called "Gideon". Siri can run shortcuts any time, even from the Lock Screen or inside another app. This one:

1. says "Yes?"
2. listens to you
3. asks Gemini (with Gideon's personality)
4. speaks the answer
5. **keeps listening for your next question**, until you say "that's all", "goodbye", or go quiet.

Then you trigger it in one of two ways:
- **"Hey Siri, Gideon"** works on any recent iPhone.
- **Just "Gideon"** works with **Vocal Shortcuts** on iOS 18 or later (see the end of this guide).

It takes about 10 minutes to build, and you only do it once. Every step happens in the **Shortcuts** app (white and pink/blue icon, already on your iPhone).

---

## Part 1: Create the shortcut

1. Open **Shortcuts**, then tap **+** (top right).
2. Tap the name at the top, choose **Rename**, and type **Gideon**.

### Step A: Gideon's personality
3. Tap **Search Actions**, type **Text**, and add the **Text** action.
4. In the Text box, paste this:

```
You are Gideon, a calm, precise, slightly formal AI assistant with a dry sense of humour. You are an original character created by Liam, and you are talking to Liam through Siri, so your reply is read aloud. Answer in 1 to 3 short sentences. No markdown, no lists, no emojis. Be kind and appropriate for a teenager. When Liam is clearly finished (thanks, that's all, goodbye, see you), reply in a few words and put [END] at the very end.
Today is
```

5. Put the cursor after "Today is ", type a space, then tap **Current Date** in the variable bar above the keyboard.
6. Add the **Set Variable** action. Set **Variable Name** to `System`. The input should already be "Text".

### Step B: Greeting
7. Add **Speak Text** and type `Yes?` into it. Tap the little **>** arrow on the action and make sure **Wait Until Finished** is ON. You can also choose a voice and speed here.

### Step C: The conversation loop
8. Add **Repeat**. Tap the number and set it to **20**. That means up to 20 back-and-forths per conversation. Put **all of the following actions inside the Repeat**, between "Repeat" and "End Repeat". Dragging them there works.

**Listen:**

9. Add **Dictate Text**. Tap **>** and set **Stop Listening** to **After Pause**. Set Language to English.
10. Add **If**. Set it to **Dictated Text** · **does not have any value**.
11. Inside that If, add **Speak Text** `Standing by.` and then **Stop This Shortcut**. Then delete the "Otherwise" part if there is one, so it reads `If … / Speak / Stop / End If`.

**Say "that's all" to finish:**

12. Add **Match Text**. Set the pattern to:

```
(?i)^\W*(ok\W*|okay\W*|thanks\W*|thank you\W*)*(that'?s all|that'?s it|that will be all|goodbye|bye|never ?mind|stop|nothing|no thanks?|i'?m done)\W*(gideon)?\W*$
```

   …and set the input text to **Dictated Text**.

13. Add **If**. Set it to **Matches** · **has any value**. Inside it, add **Speak Text** `Very well. Standing by.` and then **Stop This Shortcut**. End If.

**Remember the conversation so far:**

14. Add **Text**. Inside it, tap the variable bar and insert the **Chat** variable. The first time, type the word `Chat` in the variable picker under "Variables" or "Select Variable". It will exist after step 15. Then a new line, `Liam: `, then insert **Dictated Text**.
15. Add **Set Variable**, name it `Chat`, input **Text**.

> Tip: if Chat isn't offered yet in step 14, do step 15 first. Then go back to the Text action and insert Chat at the start.

**Ask Gemini:**

16. Add **Get Contents of URL**. In the URL box, type this (one line):

```
https://generativelanguage.googleapis.com/v1beta/models/gemini-3.8-flash:generateContent
```

17. Tap **>** (Show More) and set **Method: POST**.
18. **Headers**: tap *Add new header* twice:
    - `Content-Type` = `application/json`
    - `x-goog-api-key` = *your Gemini key*, the same one you pasted into Gideon's ⚙︎ Setup.
19. **Request Body: JSON**. Add these fields exactly. Each "Add new field" asks for a type.
    - `system_instruction`, type **Dictionary**. Inside it:
      - `parts`, type **Array**. Inside it, add one item, type **Dictionary**. Inside that:
        - `text`, type **Text**, value = the **System** variable
    - `contents`, type **Array**. Inside it, add one item, type **Dictionary**. Inside that:
      - `role`, type **Text**, value `user`
      - `parts`, type **Array**. Inside it, add one item, type **Dictionary**. Inside that:
        - `text`, type **Text**, value = the **Chat** variable

**Dig the answer out of Google's reply.** Add these actions in order. Each one automatically uses the previous result.

20. **Get Dictionary Value**: Get **Value** for key `candidates`
21. **Get Item from List**: **First Item**
22. **Get Dictionary Value**: key `content`
23. **Get Dictionary Value**: key `parts`
24. **Get Item from List**: **First Item**
25. **Get Dictionary Value**: key `text`
26. **Set Variable**: name `Reply`

**Speak it:**

27. Add **Replace Text**. Find `\[[^\]]*\]`, replace with nothing (leave it empty), in **Reply**. Tap **>** and turn **Regular Expression** ON. This hides the `[END]` tag so Siri doesn't read it out.
28. Add **Speak Text** with the **Updated Text** from step 27. Make sure **Wait Until Finished** is ON.
29. Add **Text**: insert **Chat**, new line, `Gideon: `, insert **Reply**. Then **Set Variable** `Chat` to it. Now Gideon remembers what it said.
30. Add **If**: **Reply** · **contains** · `[END]`. Inside it, add **Stop This Shortcut**. End If.

That's the end of the Repeat. After **End Repeat**, add **Speak Text** `Standing by.`

31. Tap **Done**.

### Try it
Tap the shortcut once to test it. The first time, iPhone asks permission for the microphone and to connect to `generativelanguage.googleapis.com`. Tap **Always Allow**.

Then say: **"Hey Siri, Gideon."**

---

## Part 2: Wake it by saying just "Gideon" (iOS 18+)

**Vocal Shortcuts** let you pick your own trigger phrase, so there's no "Hey Siri" needed.

1. Open **Settings**, then **Accessibility**, then **Vocal Shortcuts**.
2. Tap **Set Up Vocal Shortcuts** (or **Add Action**).
3. Choose **Shortcut**, then **Gideon**.
4. When it asks for the phrase, say **"Gideon"** three times.

Now saying "Gideon" runs the conversation shortcut from anywhere: the Home Screen, other apps, and even the Lock Screen. On the Lock Screen, iPhone may ask you to unlock before it can use the internet.

Things to know:
- One-word phrases sometimes trigger by accident. If that happens, use a phrase like **"Hey Gideon"** instead.
- Vocal Shortcuts listen on the phone itself (not Siri's servers) and use a little extra battery.
- If you don't see Vocal Shortcuts, update iOS (Settings, then General, then Software Update), or just use "Hey Siri, Gideon".

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Siri says there's a problem with the shortcut at step 20 | Gemini sent an error instead of an answer. Run the shortcut from the Shortcuts app and tap the error to read it. **404** means the model name is out of date: change `gemini-3.8-flash` in the URL to a current free Flash model. **400 API key** means the key was pasted wrong. **429** means the free limit was reached, so wait a minute. |
| It stops listening too fast | In **Dictate Text**, set **Stop Listening** to **After Short Pause**, or speak right after "Yes?". |
| It uses Siri's voice, not Gideon's | Tap **>** on each **Speak Text** and pick a voice. Download better ones in Settings, then Accessibility, then Spoken Content, then Voices. |
| Memory and timers don't work here | Those live inside the Gideon web app on your phone; the shortcut can't see them. For timers by voice, just ask Siri directly: "Hey Siri, set a timer for 5 minutes." |

**Keep your key private:** this shortcut contains your Gemini key. Don't share the shortcut with anyone. If you ever do, make a new key at aistudio.google.com and delete the old one.
