AUNTY LILY HUD: FRAMEWORK V0.1
==============================

Status: written but never compiled or run in-world. Expect a few compile errors and some behaviour that needs adjusting. Paste any error messages back and they can be fixed quickly.

WHAT'S HERE
-----------

  lsl/al_main.lsl: The HUD controller: consent flow, menus (llDialog), tasks, mood check-in, help, flags, "friend came out", settings, quiet mode, Forget Me.
  lsl/al_content.lsl: The content loader: seeds a cache from notecards, optionally refreshes it from the web, answers lookups.
  tools/build_content.py: Validates the content files and splits them into HUD-sized parts plus manifest.txt.
  build/: The output of the build tool (run on the current content, version 0.6.0).

The source content lives in aunty_lily_content_v0.1/. Edit it there, then run:

    python3 tools/build_content.py ../aunty_lily_content_v0.1 build 0.6.1

The tool refuses to write anything if a record is invalid (wrong field count, label over 24 characters, a page over 480 characters, and so on).

RUNNING THE BUILD TOOL ON YOUR COMPUTER (WINDOWS)
-------------------------------------------------

The tool only needs Python 3. It uses nothing else, so there is nothing more to install.

One-time setup

1. Download Python 3 from python.org and run the installer. Tick "Add python.exe to PATH" on the first screen, then choose Install.
2. Open Command Prompt (press the Windows key, type cmd, press Enter).
3. Type python --version and press Enter. It should print something like Python 3.12.x. If Windows says Python isn't found, try py --version. If that works, use py instead of python in the commands below.

Each time you change the content

1. Put the two folders side by side on your computer: aunty_lily_framework and aunty_lily_content_v0.1.
2. In Command Prompt, go into the framework folder. For example: cd C:\Users\YourName\Documents\aunty_lily_framework
3. Run the build, changing the last number each time (the content version):

    python tools\build_content.py ..\aunty_lily_content_v0.1 build 0.6.1

4. If it worked, it prints "Built 10 content files" and a list of the files. The build folder now holds the new files.
5. If it found a problem, it prints ERRORS with the file and line number (for example a label over 24 characters) and writes nothing. Fix that line in the content file and run it again.
6. You'll also see a count of records still marked draft, and a warning for each helpline that is only secondary-verified. Those are reminders, not errors.

Then update the HUD and the website

- Website: upload the contents of build (including manifest.txt) to the content folder on GitHub Pages. HUDs that have consented will pick up changed files on their next check.
- Offline notecards: only needed if you want new HUDs to ship with the new text. Replace the notecards in the HUD prim with the new build files (not manifest.txt), and change SEED_VERSION in al_content.lsl so the HUD refills its cache from them.

HOW IT FITS TOGETHER
--------------------

- Two scripts in one prim. They talk by link message and share Linkset Data (which any script in the linkset can read).
- User data is stored under keys starting u: (consent, mood on/off, country, quiet mode, task progress, ribbons, mood numbers). Nothing is stored unless the person agrees on the first-run screen.
- Content cache is stored under c:f:<file> (the text) and c:v:<file> (its version). Forget Me deletes every u: key and leaves the cache alone.
- Offline set: the build files are put inside the HUD as notecards. On first run al_content copies them into the cache, so the HUD works with no internet at all.
- Web updates: al_main asks al_content to check the web at most every 7 days, and only after consent. al_content also checks for consent itself before making any request. It tries the sources in order and downloads only files whose version has changed.
- Consent text is baked into al_main. Remote content can never change what someone agreed to. Keep it in step with the privacy statement, and bump CONSENT_VERSION when the statement changes enough to need fresh consent.

SETTING IT UP
-------------

1. Build the HUD. Rez one prim, name it "Aunty Lily HUD", and put al_content.lsl and al_main.lsl in it.
2. Add the offline notecards. Put every .txt file from build/ except manifest.txt into the prim, keeping the names exactly (for example aunty.1.txt). Leave them full permission while testing.
3. Edit the placeholders. In al_content.lsl, set SOURCES (the YOURNAME text means "not set up yet", and the script skips it). In al_main.lsl, set PRIVACY_URL.
4. Wear it on a HUD attachment point and touch it.
5. GitHub Pages. Create a repository, turn on Pages, and upload the contents of build/ (including manifest.txt) to a content/ folder. Put the privacy statement at privacy.html. The SOURCES entry must end with a slash.

TEST CHECKLIST (DO THESE IN ORDER)
----------------------------------

1. Both scripts compile with no errors.
2. First touch shows the four consent pages. "Tell me more" opens the privacy page. "Not for me" leaves the HUD usable and stores nothing (check there are no u: keys).
3. "Fine by me" asks about mood, then country, then shows the home menu with a quip.
4. Today's task shows task 1. Done advances to task 2 and shows a "task done" line. After task 7, the next press shows the "series done" line once and sends the ribbon message.
5. Help shows the right country's numbers, with the Website button working. Check each number against helplines.txt.
6. Flags (green and red), then "Friend came out" for each relationship, both Cards and FAQ. Pages move forward and back, and More > works on long menus.
7. Check-in stores a number; a low number offers the help menu. Settings shows mood and country correctly.
8. Forget Me wipes everything, and the next touch shows the consent screen again.
9. Quiet mode shows only the time. Wake returns. Panic wipes and hides the HUD, and re-attaching shows it again.
10. With SOURCES pointing at a real GitHub Pages site: the web refresh updates a changed file (change one record, rebuild, re-upload, and wait or clear c:last).
11. Give a copy to an alt: it must start with no u: data (a new owner wipes it).
12. Check memory: use llGetFreeMemory in a debug build, or watch for stack-heap collisions during the seed.

THINGS I COULDN'T CHECK FROM HERE (LOOK THEM UP ON THE LSL WIKI)
----------------------------------------------------------------

- That llLinksetDataFindKeys(pattern, start, count) and llLinksetDataDeleteFound(pattern, pass) behave as used (regex pattern, and that deleting by ^u: removes only user keys).
- That llGetNotecardLine returns long enough lines for the longest records (about 340 characters).
- That scripts can read the notecards at the permissions you ship them with.
- The llHTTPRequest parameter names and the request rate limit. Files are fetched one at a time, which should be well inside it.
- Whether Linkset Data survives replacing scripts in a worn HUD with a script pin (the update plan relies on it).
- That dialog text and button limits (512 characters, 24 per label, 12 buttons) are as assumed.

PACKAGING RULES (PRIVACY)
-------------------------

- Before packaging a release copy, wipe it: delete every u: key (and c:last). A released HUD must start empty.
- Make the product no-transfer if possible. The script already wipes user data if the owner changes.
- Never add logging, chat output (llSay, llOwnerSay) or any web request that includes user data. The only web requests are for content files.

KNOWN GAPS (THE TODO LIST)
--------------------------

- The Skip button on a task just shows a line; there's no "swap for another task" yet.
- The ribbon reward is only a link message (LM_RIBBON, 4001, string = series id). There is no ribbon script yet.
- Time-aware greetings (morning, afternoon, evening) need a timezone setting, so the home screen uses the quip lines for now.
- Low-mood streaks: any low number offers the help menu, but there's no "three lows in a week" logic.
- Companion creature and the quick-exit idea (see the brief) are not started.
- Stale cache files are not deleted when a file is removed from the manifest.
- In-world dropbox updates are not built. SOURCES only handles web addresses.
