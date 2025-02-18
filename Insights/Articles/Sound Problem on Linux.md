# Perplexity
When I use Linux, there is always cracking/clicking/popping (not sure which one) sound when I reboot my laptop, play videos or audio that I recorded from my device (not other videos). I searched web, they recommended disabling snd_hda_intel power saving and power_save_controller. It kind of helped to record voice properly, but these sounds are still there. When I pause or play an audio, move playback, those happen. My audio devices are AMD, but use the module I said

Based on the search results, here are some potential solutions for crackling, clicking, or popping sounds in Linux, particularly when using AMD audio devices with the `snd_hda_intel` module:

1.  **[Disable power saving for the sound card](pplx://action/followup)**:
    *   Create or modify the file `/etc/modprobe.d/alsa-base.conf` and add the line `options snd_hda_intel power_save=0`[2]. Alternatively, create a file named `/etc/modprobe.d/audio_disable_powersave.conf` with the content `options snd_hda_intel power_save=0`[8][10]. This disables power saving for the Intel HD Audio module[8].
    *   Reboot your system for the changes to take effect[8][10].
    *   If you want power saving but fewer pops, increase the delay before the card powers down by setting `power_save` to a higher value like `60` (seconds)[8].
2.  **[Disable `module-suspend-on-idle` in PulseAudio](pplx://action/followup)**:
    *   Comment out the line `load-module module-suspend-on-idle` in both `/etc/pulse/default.pa` and `/etc/pulse/system.pa`[1].
    *   Restart PulseAudio using the command `systemctl restart --user pulseaudio`[6].
3.  **[Check hardware and connections](pplx://action/followup)**:
    *   Ensure speakers or headphones are correctly plugged in[3].
    *   Inspect audio cables for damage by testing them with another device[3].
4.  **[Update Kernel](pplx://action/followup)**:
    *   If you have upgraded to kernel version `5.13.0-37`, there might be a bug causing audio crackling. In that case, change Pulse Audio's default sample rate to 48000[7].
5.  **[Investigate sound card drivers](pplx://action/followup)**:
    *   Some sound cards may not have good Linux drivers[3]. Search the internet for your sound card model and "Linux" to see if others have the same problem[3].
    *   Use the `lspci` command to get more information about your sound card[3].
6.  **[Other considerations](pplx://action/followup):**
    *   The issue could be related to the transition between IDLE and SUSPENDED states of the PulseAudio default sink[1].
    *   Weak power input can cause the computer to switch between charging and not charging, leading to clicking sounds[2].
    *   The popping sound may be due to the power management of the sound codec[6].
    *   For a Realtek ALC295 codec, the audio issue might be specific to this codec, and a kernel patch may not completely eliminate the problem[10].
    *   If using PipeWire, a drop-in lua file for WirePlumber might help[10].

Citations:
[1] My audio is making horrible crackling sounds — what should I do? https://discussion.fedoraproject.org/t/my-audio-is-making-horrible-crackling-sounds-what-should-i-do/76981
[2] Periodic clicking sound from PC speaker https://askubuntu.com/questions/175602/periodic-clicking-sound-from-pc-speaker
[3] I hear crackling or buzzing when sounds are playing https://help.ubuntu.com/stable/ubuntu-help/sound-crackle.html.en
[4] Arch Linux https://bbs.archlinux.org/viewtopic.php?id=200978
[5] Popping/clicking noise from speakers at times https://discussion.fedoraproject.org/t/popping-clicking-noise-from-speakers-at-times/73910
[6] How to disable the power saving for snd_hda_codec_realtek? https://unix.stackexchange.com/questions/565886/how-to-disable-the-power-saving-for-snd-hda-codec-realtek
[7] Annoying click/popping sound on Ubuntu 20.04 https://askubuntu.com/questions/1230833/annoying-click-popping-sound-on-ubuntu-20-04
[8] Stop audio pops on Intel HD Audio https://major.io/p/stop-audio-pops-on-intel-hd-audio/
[9] 10 Ways to Fix Audio/Sound Crackling or Popping on Windows https://repairit.wondershare.com/audio-repair/fix-audio-sound-crackling-or-popping-on-windows.html
[10] Audio popping sound when playback is started or stopped https://discussion.fedoraproject.org/t/audio-popping-sound-when-playback-is-started-or-stopped/100879

# Claude
This is a common issue with Linux audio, particularly with the snd_hda_intel driver, which is used for many audio controllers including some AMD ones. Let me help you troubleshoot this.

The clicking/popping sound you're hearing is likely due to power management features turning the audio codec on and off. While you've already tried disabling some power saving features, there are a few more things we can try:

1. First, let's create a more comprehensive fix for the power management settings. Create this file:

```bash
sudo nano /etc/modprobe.d/audio_disable_powersave.conf
```

Add these lines:
```
options snd_hda_intel power_save=0
options snd_hda_intel power_save_controller=N
options snd_hda_intel enable_msi=1
```

2. You might also want to modify PulseAudio settings. Create or edit:

```bash
sudo nano /etc/pulse/daemon.conf
```

Look for and modify these lines (or add them if they don't exist):
```
enable-deferred-volume = no
default-sample-rate = 48000
alternate-sample-rate = 44100
```

1. If you're using PipeWire instead of PulseAudio, try creating:

```bash
nano ~/.config/pipewire/pipewire.conf.d/99-input-declick.conf
```

With:
```
context.properties = {
    default.clock.rate = 48000
    default.clock.quantum = 1024
    default.clock.min-quantum = 1024
    default.clock.max-quantum = 2048
}
```

After making these changes, reboot your system. If you're still experiencing issues, could you tell me:
2. Which Linux distribution you're using?
3. Are you using PulseAudio or PipeWire?
___
Ah, that helps narrow it down. Since you're using Fedora with PipeWire, let's try some PipeWire-specific solutions:

1. First, let's check your current PipeWire configuration. Run:
```bash
systemctl --user status pipewire pipewire-pulse
```

2. Create a dedicated config directory if it doesn't exist:
```bash
mkdir -p ~/.config/pipewire/pipewire.conf.d/
```

1. Create a custom configuration file:
```bash
nano ~/.config/pipewire/pipewire.conf.d/99-custom-settings.conf
```

Add these optimized settings:
```
context.properties = {
    default.clock.rate = 48000
    default.clock.quantum = 1024
    default.clock.min-quantum = 1024
    default.clock.max-quantum = 2048
    core.daemon = true
    core.clock.power-of-two-quantum = true
}

context.modules = [
    { name = libpipewire-module-rt
        args = {
            nice.level = -11
            rt.prio = 88
            rt.time.soft = -1
            rt.time.hard = -1
        }
        flags = [ ifexists nofail ]
    }
]
```

2. Also, let's try adjusting your ALSA settings. Create:
```bash
sudo nano /etc/modprobe.d/alsa-base.conf
```

Add:
```
options snd_hda_intel buffer_size=32768
options snd_hda_intel enable_msi=1
options snd_hda_intel single_cmd=1
options snd_hda_intel probe_mask=1
```

3. After making these changes, restart PipeWire:
```bash
systemctl --user restart pipewire pipewire-pulse
```

If the issue persists, we can try:
4. Checking your current audio buffer status with:
```bash
pw-top
```
This will help us see if there are any buffer underruns or other issues.

Would you like to try these changes first and let me know if they help? Also, could you tell me your specific AMD audio hardware model? We might need to add some model-specific parameters.
# Copilot
It sounds like you're experiencing audio distortion issues, which can be quite frustrating! Since disabling `snd_hda_intel` power saving helped a bit but didn't completely resolve the issue, there are a few more steps you can try:

1. **Update your audio drivers**: Ensure that your AMD audio drivers are up to date[43dcd9a7-70db-4a1f-b0ae-981daa162054](https://www.minitool.com/news/amd-high-definition-audio-device.html?citationMarker=43dcd9a7-70db-4a1f-b0ae-981daa162054 "1"). You can download the latest drivers from the AMD website[43dcd9a7-70db-4a1f-b0ae-981daa162054](https://www.minitool.com/news/amd-high-definition-audio-device.html?citationMarker=43dcd9a7-70db-4a1f-b0ae-981daa162054 "1").

2. **Check for BIOS updates**: Sometimes, updating your BIOS can resolve compatibility issues with hardware[43dcd9a7-70db-4a1f-b0ae-981daa162054](https://community.amd.com/t5/pc-graphics/amd-high-definition-audio-device-missing-in-quot-sounds-quot-but/td-p/699483?citationMarker=43dcd9a7-70db-4a1f-b0ae-981daa162054 "2"). Check if there's a BIOS update available for your motherboard and install it if needed[43dcd9a7-70db-4a1f-b0ae-981daa162054](https://community.amd.com/t5/pc-graphics/amd-high-definition-audio-device-missing-in-quot-sounds-quot-but/td-p/699483?citationMarker=43dcd9a7-70db-4a1f-b0ae-981daa162054 "2").

3. **Disable power-saving features**: You've already done this, but make sure it's set correctly[43dcd9a7-70db-4a1f-b0ae-981daa162054](https://www.reddit.com/r/Fedora/comments/psez5e/turning_off_power_saving_option_on_intel_sound/?citationMarker=43dcd9a7-70db-4a1f-b0ae-981daa162054 "3"). Edit the file `/etc/modprobe.d/alsa-base.conf` and add the following line:
   ```
   options snd-hda-intel power_save=0 power_save_controller=N
   ```
   Then reboot your system[43dcd9a7-70db-4a1f-b0ae-981daa162054](https://www.reddit.com/r/Fedora/comments/psez5e/turning_off_power_saving_option_on_intel_sound/?citationMarker=43dcd9a7-70db-4a1f-b0ae-981daa162054 "3").

4. **Check for latency issues**: High latency can cause audio popping and clicking[43dcd9a7-70db-4a1f-b0ae-981daa162054](https://community.amd.com/t5/pc-processors/high-latency-causing-skipping-audio-problems-only-happens-with/m-p/598091?citationMarker=43dcd9a7-70db-4a1f-b0ae-981daa162054 "4"). Use a tool like LatencyMon to check for any latency issues and see if any drivers are causing high latency[43dcd9a7-70db-4a1f-b0ae-981daa162054](https://community.amd.com/t5/pc-processors/high-latency-causing-skipping-audio-problems-only-happens-with/m-p/598091?citationMarker=43dcd9a7-70db-4a1f-b0ae-981daa162054 "4").

5. **Try a different kernel**: Sometimes, switching to a different kernel version can resolve audio issues. You can try installing a different kernel and see if it helps.

6. **Use a different audio server**: If you're using PulseAudio, you might want to try switching to PipeWire or ALSA to see if it makes a difference.

If none of these steps work, it might be worth reaching out to the Linux audio community or forums for more specific advice. Let me know if you need any more help or if you have any other questions!
___
I'm sorry those suggestions didn't resolve your issue. Let's try a more in-depth approach. Here are a few additional troubleshooting steps to consider:

1. **Check Audio Configuration Files**:
   - Sometimes, configuration files can cause issues. Look into `/etc/pulse/daemon.conf` and `/etc/pulse/default.pa` if you are using PulseAudio. Adjust the parameters related to latency:
     ```
     default-fragments = 5
     default-fragment-size-msec = 2
     ```
   - If you're using ALSA, check `/etc/asound.conf` or `~/.asoundrc` for any unusual settings.

2. **Test with Live USB**:
   - Boot your laptop from a Linux live USB to see if the issue persists. This helps to determine if it's a system-wide issue or specific to your installation.

3. **Inspect Logs**:
   - Check system logs for any errors related to audio. You can use the following command:
     ```
     sudo dmesg | grep snd
     ```
   - Also, check PulseAudio logs if you're using it:
     ```
     pulseaudio --start --log-target=syslog
     ```
   - Review any entries that might provide clues.

4. **Install `pavucontrol`**:
   - Install the PulseAudio Volume Control tool with:
     ```
     sudo apt-get install pavucontrol
     ```
   - Open it and check the configuration tabs to ensure everything looks correct. Sometimes, tweaking settings here can resolve issues.

5. **Consider IRQ Conflicts**:
   - Audio issues can sometimes be caused by IRQ conflicts. You can check your IRQ settings using:
     ```
     cat /proc/interrupts
     ```
   - Look for any conflicts or high usage on IRQs related to your audio devices.

6. **Experiment with Power Management Options**:
   - Sometimes, adjusting power management settings in the BIOS/UEFI can help. Look for settings related to CPU and device power management.

If you're still having trouble, you might want to seek help from the Linux audio community or forums where users with similar hardware might have faced and resolved the same issue. Let me know how it goes!
# Gemini
You're on the right track with investigating the snd_hda_intel module and power-saving features, as those are often the culprits for these kinds of audio issues on Linux, especially with AMD audio hardware.  Even though disabling power saving helped somewhat, the persistent clicks/pops indicate there's still something interfering.  Let's troubleshoot this further.
Here's a breakdown of things to try, going from most likely to less common causes:
1. Further Power Management Tweaks:
 * snd_hda_intel module parameters:  While you might have disabled the main power saving, there are more granular options. Try these:
   * power_save=0:  This should be the primary one you've tried, but double-check it's applied correctly.
   * position_fix=1: This sometimes helps with clicks and pops related to audio stream positioning.
   * enable_msi=1:  (Message Signaled Interrupts) Can sometimes improve performance and reduce glitches.  However, some systems don't work well with this, so try it and if the problem worsens or your audio breaks entirely, remove it.
   To apply these, you'll need to edit your kernel boot parameters.  The exact method varies by distribution (GRUB, systemd-boot, etc.). Look up how to "add kernel boot parameters" for your specific distro.  A common way is to edit /etc/default/grub (for GRUB), add the parameters to the GRUB_CMDLINE_LINUX_DEFAULT line, and then update GRUB (e.g., sudo update-grub).
 * Check other power-saving settings:  Your system might have other power-saving features affecting the audio.  Look in your desktop environment's power management settings.  Also, check for any settings related to "audio power saving" or "sound card power management."
2. ALSA Configuration:
 * /etc/asound.conf or ~/.asoundrc:  These files control ALSA (Advanced Linux Sound Architecture).  Sometimes, misconfigurations here can cause problems.  Try temporarily renaming these files (if they exist) to see if it makes a difference. This will help determine if they are the cause.
 * alsamixer:  Run alsamixer in a terminal.  Check all the volume levels and make sure nothing is muted or excessively high.  Pay particular attention to anything related to "PCM," "Master," and "Front."  Also, look for any unusual settings.  Sometimes, just toggling things (even if they seem correct) can resolve glitches.
3. Driver Updates (Less Likely, but Possible):
 * Kernel updates: Ensure you have the latest kernel for your distribution.  Newer kernels often have updated drivers.
 * Firmware updates: Check if there are any firmware updates available for your motherboard or audio hardware.  This is less common but can sometimes be necessary.  Your motherboard manufacturer's website is the best place to look.
4. Hardware Issues (Least Likely):
 * Cables/Connections:  While less likely for internal laptop speakers, if you're using external speakers or headphones, double-check all connections.
 * Hardware failure:  In rare cases, the sound card itself might be failing.  This is usually accompanied by more severe audio problems than just clicks and pops.
5.  Specific to your recorded audio:
 * Recording settings: It's worth double-checking the recording settings you used when capturing the audio.  A very high sample rate or bit depth might be stressing the system, although this is less likely to cause clicks and pops and more likely to cause distortion.
 * Software used for recording:  Try a different recording application to see if the problem persists.  This can help isolate whether the issue is with the recording software or the audio system itself.
Debugging Steps:
 * Isolate the sound:  Try to pinpoint exactly when the sound occurs.  Is it only when playing your recorded audio?  Only during boot?  Only when certain applications are running?  This will help narrow down the cause.
 * Check system logs: Look in your system logs (e.g., /var/log/syslog or similar, depending on your distribution) for any error messages related to audio.  These logs might provide clues.
 * Boot a live CD/USB:  Boot your laptop from a live Linux distribution (like Ubuntu or Fedora).  If the problem doesn't occur in the live environment, it points to a configuration issue in your regular installation.  If it does occur, it might be a hardware problem or a driver issue that's common to your hardware.
Example for GRUB (Adding Kernel Parameters):
 * sudo nano /etc/default/grub
 * Find the line GRUB_CMDLINE_LINUX_DEFAULT="..."
 * Add the parameters inside the quotes, e.g., GRUB_CMDLINE_LINUX_DEFAULT="quiet splash power_save=0 position_fix=1"
 * Save the file.
 * sudo update-grub
 * Reboot.
Remember to test each change individually so you know what's working and what's not.  If something makes the audio worse, remove the change.  Provide details about your Linux distribution, kernel version, and the specific AMD audio hardware you have for more targeted advice.
# Deepseek
Great! Since you're already using **PipeWire**, which is known for its improved handling of audio compared to PulseAudio, we can focus on optimizing your PipeWire setup to address the cracking/clicking/popping sounds. Here are some steps tailored for PipeWire:

---

### 1. **Check PipeWire Configuration**
   - PipeWire uses configuration files to manage audio settings. You can tweak these to improve performance:
     - Open the PipeWire configuration file:
       ```bash
       sudo nano /usr/share/pipewire/pipewire.conf
       ```
     - Look for the `default.clock.rate` and `default.clock.quantum` settings. Adjust them as follows:
       ```
       default.clock.rate = 48000
       default.clock.quantum = 1024
       ```
       - `default.clock.rate`: Set this to match your audio device's sample rate (e.g., 48000 Hz is common).
       - `default.clock.quantum`: Increase this value to reduce buffer underruns (e.g., 1024 or 2048).
     - Save the file and restart PipeWire:
       ```bash
       systemctl --user restart pipewire.service
       ```

---

### 2. **Adjust Buffer Sizes**
   - PipeWire allows you to adjust buffer sizes to reduce audio glitches:
     - Edit the PipeWire client configuration file:
       ```bash
       sudo nano /usr/share/pipewire/client.conf
       ```
     - Look for the `default.pw.min-quantum` and `default.pw.max-quantum` settings. Adjust them as follows:
       ```
       default.pw.min-quantum = 1024
       default.pw.max-quantum = 2048
       ```
     - Save the file and restart PipeWire:
       ```bash
       systemctl --user restart pipewire.service
       ```

---

### 3. **Disable Power Saving for Audio Devices**
   - Even with PipeWire, power-saving features for audio devices can cause issues. Ensure they are disabled:
     - Create or edit the following file:
       ```bash
       sudo nano /etc/modprobe.d/audio_disable_powersave.conf
       ```
     - Add the following lines:
       ```
       options snd_hda_intel power_save=0 power_save_controller=N
       ```
     - Save the file and update the initramfs:
       ```bash
       sudo update-initramfs -u
       ```
     - Reboot your system.

---

### 4. **Check for Real-Time Scheduling**
   - PipeWire uses real-time scheduling to prioritize audio processing. Ensure it is enabled:
     - Check the PipeWire logs for real-time scheduling:
       ```bash
       journalctl --user -u pipewire
       ```
     - If real-time scheduling is not enabled, you may need to adjust your user's permissions:
       - Add your user to the `audio` and `realtime` groups:
         ```bash
         sudo usermod -aG audio,realtime $USER
         ```
       - Reboot your system.

---

### 5. **Test with WirePlumber**
   - PipeWire often works with **WirePlumber** as its session manager. Ensure WirePlumber is installed and running:
     - Install WirePlumber:
       ```bash
       sudo apt install wireplumber
       ```
     - Restart PipeWire and WirePlumber:
       ```bash
       systemctl --user restart pipewire.service wireplumber.service
       ```

---

### 6. **Check for Conflicting Audio Servers**
   - Ensure no other audio servers (e.g., PulseAudio) are running alongside PipeWire:
     - Stop and disable PulseAudio:
       ```bash
       systemctl --user stop pulseaudio.service
       systemctl --user disable pulseaudio.service
       ```
     - Ensure PipeWire is the default audio server:
       ```bash
       systemctl --user enable pipewire.service pipewire-pulse.service
       ```

---

### 7. **Update PipeWire and Dependencies**
   - Ensure you are using the latest version of PipeWire and its dependencies:
     - Update your system:
       ```bash
       sudo apt update
       sudo apt upgrade
       ```
     - If your distribution's package manager does not provide the latest version, consider installing PipeWire from source or a PPA.

---

### 8. **Test with a Different Audio Device**
   - If the issue persists, test with an external USB sound card or headphones to rule out hardware issues with your laptop's internal audio device.

---

### 9. **Check PipeWire Logs**
   - PipeWire logs can provide insights into what might be causing the issue:
     - View the logs:
       ```bash
       journalctl --user -u pipewire
       ```
     - Look for errors or warnings related to buffer underruns, sample rates, or hardware issues.

---

### 10. **Consider Kernel Tweaks**
   - If the issue persists, you can try adjusting kernel parameters to reduce audio glitches:
     - Edit the GRUB configuration file:
       ```bash
       sudo nano /etc/default/grub
       ```
     - Add the following parameters to the `GRUB_CMDLINE_LINUX_DEFAULT` line:
       ```
       intel_idle.max_cstate=1 processor.max_cstate=1
       ```
     - Save the file and update GRUB:
       ```bash
       sudo update-grub
       ```
     - Reboot your system.

---

By following these steps, you should be able to resolve the cracking/clicking/popping sounds while using PipeWire. Let me know if you need further assistance!
# ChatGPT
It sounds like you're dealing with audio crackling/popping issues related to the snd_hda_intel module's power-saving features, which is a common issue on Linux. Since you’ve already disabled power_save and power_save_controller, but the issue persists when pausing/playing or seeking audio, let’s try a few more steps to fix it.

Steps to Fix the Crackling/Popping Audio Issue:

1. Ensure Power Saving is Completely Disabled

Since you've already tried disabling power saving, let's confirm it’s completely turned off.

Edit (or create) this file:

sudo nano /etc/modprobe.d/alsa-base.conf

Add the following line:

options snd_hda_intel power_save=0 power_save_controller=N

Save and exit (CTRL+X, then Y, then Enter).
Then, update initramfs and reboot:

sudo dracut -f
sudo reboot

This ensures the settings persist.

2. Adjust PulseAudio or PipeWire Settings

Depending on whether you use PulseAudio or PipeWire, modifying buffer settings can help.

For PulseAudio Users: Edit this file:

sudo nano /etc/pulse/daemon.conf

Find (or add) these lines:

default-fragments = 5
default-fragment-size-msec = 5
resample-method = speex-float-5

Then restart PulseAudio:

pulseaudio -k && pulseaudio --start

For PipeWire Users (Fedora defaults to PipeWire): Edit PipeWire’s config:

sudo nano /etc/pipewire/pipewire.conf

Find the context.properties section and add:

default.clock.rate = 48000
default.clock.quantum = 1024
default.clock.min-quantum = 512

Then restart PipeWire:

systemctl --user restart pipewire pipewire-pulse

1. Try Forcing a Different Audio Profile

Check available profiles:

pactl list cards

Then switch profiles (replace <profile> with an available one):

pactl set-card-profile 0 <profile>

For example, try pro-audio mode:

pactl set-card-profile 0 pro-audio

2. Kernel Parameter Fix

Some AMD audio issues can be fixed by adding a kernel parameter.

Edit GRUB:

sudo nano /etc/default/grub

Find the line:

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"

Modify it to:

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash snd_hda_intel.enable_msi=1"

Then update GRUB and reboot:

sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo reboot


---

Final Thoughts

Since you’re using Fedora KDE with AMD audio, the issue could also be related to PipeWire (which replaced PulseAudio in Fedora). If these fixes don’t help, consider temporarily switching back to PulseAudio (dnf swap pipewire-pulseaudio pulseaudio --allowerasing) to see if it makes a difference.

Check if the issue happens only on battery or also when plugged in. Some aggressive power management settings could be affecting audio.

If it persists, test with a Live USB of another distro (e.g., Ubuntu) to see if the issue is Fedora-specific.


Try these and let me know if any of them help!
____
Since the issue persists across all distros, that suggests it's not Fedora-specific but rather a kernel, driver, or hardware issue with how Linux handles your AMD audio device.

Answering Your Questions:

1. Why dracut?

You don’t actually need dracut unless you want to regenerate the initramfs on Fedora (since Fedora doesn’t use update-initramfs like Ubuntu/Debian). However, for most kernel module changes, a simple reboot is enough.

2. Why the second option (PulseAudio/PipeWire settings)?

Since Fedora uses PipeWire by default, tweaking buffer sizes and sample rates can help prevent crackling and popping caused by low-latency settings that don’t work well with some AMD hardware.

If PipeWire doesn’t handle your hardware well, reverting to PulseAudio can be a useful test, because some AMD users report better stability with PulseAudio over PipeWire.



---

More Fixes to Try

Since the issue happens across all distros, let's try a few more things:

1. Check DPC Latency (Common AMD Issue)

If your system has high Deferred Procedure Call (DPC) latency, it can cause crackling and popping.

Run:

cat /sys/kernel/debug/dri/0/amdgpu_pm_info

Look for "PCIe L1/L0 transitions"—if it's happening often, your GPU/CPU might be throttling power too aggressively, which can mess with audio.

To test if it's power management related, try:

sudo echo on > /sys/bus/pci/devices/0000:XX:XX.X/power/control

Replace 0000:XX:XX.X with your AMD audio device ID, found with:

lspci | grep -i audio

If this fixes the issue, you can make it permanent.

2. Force ALSA to Use a Specific Driver

Sometimes the wrong driver/module is loaded for AMD audio.

Check which module is in use:

lsmod | grep snd

Try forcing the correct one:

echo "options snd-hda-intel model=auto" | sudo tee -a /etc/modprobe.d/alsa-base.conf
sudo reboot

1. Check for IRQ Conflicts

Crackling can happen if your audio and GPU share the same IRQ (interrupt request).

Run:

cat /proc/interrupts | grep -i snd

If your sound card shares an IRQ with another device (especially the GPU), this could be causing the issue.
A workaround is to disable MSI (Message Signaled Interrupts) for your audio device:

echo "options snd-hda-intel enable_msi=0" | sudo tee -a /etc/modprobe.d/alsa-base.conf
sudo reboot
___
Next Steps

1. Run the DPC latency check (amdgpu_pm_info)
2. Try disabling PCIe power-saving
3. Check if IRQ conflicts exist
4. Test forcing ALSA settings

Let me know what you find!

