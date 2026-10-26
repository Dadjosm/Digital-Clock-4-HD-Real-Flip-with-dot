# Digital Clock 4 - flip animation build

Builds `digital_clock.exe` (v4.7.8 + flip animation for skins with "flip" in the name)
on GitHub's servers. Nothing to install.

1. Put these files in a new GitHub repo (keep the paths exactly):
   - `.github/workflows/build.yml`
   - `flip-animation.patch`
2. Open the repo's **Actions** tab, pick **Build Digital Clock (flip animation)**, press **Run workflow**.
3. When it turns green (about 10 minutes), open the run, download the artifact
   **digital_clock-flip-exe** and unzip it.
4. Close Digital Clock, copy the new `digital_clock.exe` over the old one in your
   `Digital Clock 4` folder, and start it. Choose the "Flipclock HD" skin.
