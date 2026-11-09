# Talk to Gideon without opening the app

**Why a shortcut?** iPhone doesn't let *any* website use the microphone while it's closed or the screen is locked. Apple blocks that for privacy. Siri *can* run a **Shortcut** any time, even from the Lock Screen or inside another app. So we build a shortcut called **Gideon** that:

1. says "Yes?"
2. listens to you
3. asks Gemini, with Gideon's personality, memories and to-do list
4. speaks the answer
5. **keeps listening for your next question**, until you say "that's all" or go quiet.

Then you start it with:
- **"Hey Siri, Gideon"** (any iPhone), or
- **just "Gideon"**, using Vocal Shortcuts (iOS 18 or later, Part G).

It takes about 15 minutes, once. You'll switch between **Gideon** (in Safari) and the **Shortcuts** app, which is already on your iPhone.

> 💡 **Copy buttons:** in Gideon, open ⚙︎ Setup and scroll down to *"Use Gideon without opening this app"*. The buttons **COPY SIRI PERSONALITY**, **COPY GEMINI LINK** and **COPY MY KEY** copy the long bits, so you only have to paste them.

---

## Part A: Start the shortcut (1 min)
1. Open **Shortcuts** and tap **+** (top right).
2. Tap the name at the top, then **Rename**, and type `Gideon`. Tap **Done** on the keyboard.

## Part B: Gideon's personality (2 min)
3. In Gideon: ⚙︎ Setup → **COPY SIRI PERSONALITY**.
4. In Shortcuts: tap **Search Actions**, type `Text`, and tap the **Text** action.
5. Tap inside the Text box and **Paste**.
6. The text ends with "Today is ". Put the cursor right at the end, then tap **Current Date** in the bar above the keyboard.
7. Search for and add **Set Variable**. Tap *Variable Name* and type `System`.

## Part C: Greeting (1 min)
8. Add **Speak Text**. Tap the blue *Text* and type `Yes?`.
9. Tap the small **›** arrow on that action and make sure **Wait Until Finished** is **on**. You can pick a voice here too.

## Part D: Listen, with "that's all" to finish (4 min)
10. Add **Repeat**. Tap the number and set it to `20`. **Everything from step 11 to step 30 goes inside the Repeat** (between "Repeat" and "End Repeat"). New actions normally land at the bottom, so drag each one up into the Repeat.
11. Add **Dictate Text**. Tap **›** and set **Stop Listening** to **After Pause**.
12. Add **If**. Set the condition to **Dictated Text** · **does not have any value**.
13. Inside that If, add **Speak Text** with `Standing by.`, then add **Stop This Shortcut**. If there's an "Otherwise" section, leave it empty.
14. *(After "End If".)* Add **Match Text**. Tap the pattern and paste this:

```
(?i)^\W*(ok\W*|okay\W*|thanks\W*|thank you\W*)*(that'?s all|that'?s it|that will be all|goodbye|bye|never ?mind|stop|nothing|no thanks?|i'?m done)\W*(gideon)?\W*$
```

    Set the text it searches to **Dictated Text**.
15. Add **If**. Set it to **Matches** · **has any value**. Inside it, add **Speak Text** with `Very well. Standing by.`, then **Stop This Shortcut**.

## Part E: Remember the conversation (2 min)
16. *(After that End If.)* Add **Text**. Inside it, insert the **Chat** variable (tap the variable bar, choose *Select Variable*, then **Chat**), go to a new line, type `Liam: `, and insert **Dictated Text**.
    > Chat won't be in the list yet. Do step 17 first, then come back and insert Chat at the start of this Text.
17. Add **Set Variable**, name it `Chat`, with input **Text**.

## Part F: Ask Gemini and speak the answer (6 min)
18. In Gideon, tap **COPY GEMINI LINK**. In Shortcuts, add **Get Contents of URL**, tap the URL and **Paste**.
19. Tap **›** (Show More) and set **Method** to **POST**.
20. **Headers**: tap *Add new header* twice.
    - Key `Content-Type`, Text `application/json`
    - Key `x-goog-api-key`, Text: in Gideon tap **COPY MY KEY**, then **Paste** here.
21. Set **Request Body** to **JSON** and add these fields. Each "Add new field" asks for a type:
    - `system_instruction`, type **Dictionary**. Inside it, add:
      - `parts`, type **Array**. Inside it, add one item of type **Dictionary**, and inside that:
        - `text`, type **Text** = the **System** variable
    - `contents`, type **Array**. Inside it, add one item of type **Dictionary**, and inside that:
      - `role`, type **Text** = `user`
      - `parts`, type **Array**. Inside it, add one item of type **Dictionary**, and inside that:
        - `text`, type **Text** = the **Chat** variable
22. Now dig the answer out of Google's reply. Add these in order; each one uses the result of the one before:
    - **Get Dictionary Value**: key `candidates`
    - **Get Item from List**: **First Item**
    - **Get Dictionary Value**: key `content`
    - **Get Dictionary Value**: key `parts`
    - **Get Item from List**: **First Item**
    - **Get Dictionary Value**: key `text`
23. Add **Set Variable** named `Reply`.
24. Add **Replace Text**. Find `\[[^\]]*\]`, replace with nothing (leave it empty), in **Reply**. Tap **›** and turn **Regular Expression** on. This hides tags like `[END]` so Siri doesn't read them out.
25. Add **Speak Text** with the **Updated Text**, and make sure **Wait Until Finished** is on.
26. Add **Text**: insert **Chat**, a new line, `Gideon: `, then insert **Reply**.
27. Add **Set Variable** `Chat` (input: that Text).
28. Add **If**: **Reply** · **contains** · `[END]`. Inside it, add **Stop This Shortcut**.

29. After **End Repeat** (the very bottom), add **Speak Text** with `Standing by.`
30. Tap **Done**. 🎉

### Try it
Tap the **Gideon** shortcut once. The first time, iPhone asks to use the microphone and to connect to `generativelanguage.googleapis.com`. Tap **Always Allow**. Then try **"Hey Siri, Gideon"**.

---

## Part H: Let Gideon control your phone (music, email, texts, directions, apps)

Gideon can't press buttons on your phone, but it can hand back a special **link** that opens the right app with everything filled in, for example `spotify:search:Imagine%20Dragons` or `mailto:?subject=Hi`. These steps make the shortcut open that link.

> First, in Gideon tap **COPY SIRI PERSONALITY** again and paste it over the big Text block at the top of the shortcut. The new personality teaches Gideon these links.

Add these **inside the Repeat**, right **after "Set variable Reply"** and before "Replace":
1. **Match Text**: tap the pattern and paste `\[OPEN:\s*([^\]]+)\]`. Set the text it searches to **Reply**.
2. **If**: **Matches** · **has any value**.
3. Inside that If:
   - **Get Group from Matched Text**: set it to **Group At Index 1**.
   - **Open URLs**: set it to the **Text from Group** (it should fill in automatically).
4. Leave **Otherwise** empty.

Now try: *"Gideon, play Imagine Dragons on Spotify"*, *"write an email to my teacher saying I'll be late"*, *"directions to the CN Tower"* or *"open my documents"*.

Things to know:
- iPhone always lets **you** press the final **Send**, **Call** or **Play**. Gideon gets everything ready, but it can't send or call by itself. That's an Apple safety rule.
- If your phone is locked, it asks you to unlock before opening the app.
- It can open the **Files** or **Google Docs** app, but not a specific document by name.

---

## Part G: Wake it by saying just "Gideon" (iOS 18+)
1. **Settings** → **Accessibility** → **Vocal Shortcuts**.
2. Tap **Set Up Vocal Shortcuts** (or **Add Action**), then **Shortcut**, then **Gideon**.
3. Say **"Gideon"** three times when asked.

Now saying "Gideon" starts the conversation from anywhere. On the Lock Screen, iPhone may ask you to unlock before it can use the internet.

Things to know:
- If it triggers by accident, use **"Hey Gideon"** instead.
- Vocal Shortcuts listens on the phone itself, which uses a little extra battery.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Error at "Get Dictionary Value" | Gemini sent an error instead of an answer. Run the shortcut from the Shortcuts app and tap the error to read it. **404** means the model name changed: tap **COPY GEMINI LINK** again and re-paste the URL. **400 API key** means re-paste the key. **429** means the free limit for today is used up. |
| Stops listening too fast | In **Dictate Text**, set **Stop Listening** to **After Short Pause**, and start talking right after "Yes?". |
| Doesn't know my latest memories or list | Tap **COPY SIRI PERSONALITY** again and re-paste it into the Text action from Part B. |
| Robotic voice | Tap **›** on each **Speak Text** and choose a voice. You can download better ones in Settings → Accessibility → Spoken Content → Voices. |
| Timers | The shortcut can't use Gideon's timers. Just say "Hey Siri, set a timer for 5 minutes." |

**Keep it private:** this shortcut contains your Gemini key, so don't share it. If you ever do, delete that key at aistudio.google.com and make a new one.
