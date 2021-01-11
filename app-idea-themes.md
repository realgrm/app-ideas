Posted on https://discourse.flathub.org/t/creation-of-gtk-theme-manager-app/1041

Project information:
Request for someone to develop an app capable of: show all installed themes, install other themes, preview a theme, apply a theme, schedule a change of theme, toggle between themes?, open apps with different themes. More information bellow

Name: GTK Theme Manager
Homepage: N/A
License: GPL version 3
Upstream has been contacted: N/A

---
# GTK theme manager

# Goals

This app would be capable of:

- Show all installed themes

- Install other themes

- Preview theme

- Apply theme

- Schedule a change of theme

  Example: Apply dark theme near to bed time and light theme near wake up time

- Toggle between themes?

  Example:Toggle between light and dark mode

- Open apps with different themes

  Useful for those apps that do not get along with applied themes or it feels better with a specific theme

## Characteristics

- Written using GTK4

- libhandy?

- as minimal permissions as possible

---
# Ideas

Bellow are some ideas on how it could be accomplished.

# Permissions

- No internet or other privacy related permissions

- File System Permissions:

  - to install themes: `$HOME/.themes`

  - to look for themes to be installed: `$HOME/Downloads`

  - to look for desktop files:

    - `/usr/share/applications`

    - `$HOME/.local/share/applications`

    - `/var/lib/flatpak/exports/share/applications/`

    - `/var/lib/snapd/desktop/applications/`

    - to the file to edit CRONTAB

# Theme scheduler

Themes applied based on time. It would edit the crontab file

## Default schedule

- Day/night themes

### Examples:

Everyday at 10 pm change theme to dark

```

0 22 * * * gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita-dark' >/dev/null 2>&1

```

Everyday at 06 am change theme to light

```

0 6 * * * gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita' >/dev/null 2>&1

```

The way that I can think to edit the cron is editing (with administrator permission) the file

/var/spool/cron/$USER

To avoid privilege elevation, maybe the fact that the command `crontab -e` creates temporary copy of the cron file at /temp/crontab.[RandomCode] can be used.

Maybe using nano? Or editing the temporary file? Better solution?

Be careful to the solution do not erase other cron tasks not related to this app

Maybe using a set of string to delimit the beginning and the end of this app code. So every time the it needs the information, the code between these strings is extracted, filtered to only lines that respect the rules bellow (in case someone edits the wrong place), and placed at the beginning/end of the code

### Rules:

Line equal: `# String delimiting the beginning of the code of theme app`

Line contains: `gsettings set org.gnome.desktop.interface gtk-theme`

Line equal: `# String delimiting the end of the code of theme app`

## Add custom schedule

Option to the user add another schedules

The code can be something like:

```

$minute $hour * * * gsettings set org.gnome.desktop.interface gtk-theme '[theme-name]' >/dev/null 2>&1

```

Maybe an advanced option to user edit this code manually?

# Install theme from file

After install, prompt the user with an option to audit the created file

Some code can be copied from the Pling Store

https://www.opencode.net/dfn2/pling-store-development

Or maybe the whole AppImage can be inside the app and it would be called when clicking a button.

For the first releases, a simple link to download of Pling Store or to a tutorial to how to install a theme can be useful

# Apply theme

This code can be used

```

#!/usr/bin/env bash

gsettings set org.gnome.desktop.interface gtk-theme \'$theme\'

```

# Preview theme

Opens a default window using the prefix ` GTK_THEME=$theme:$variant`

### Example:

```

#!/usr/bin/env bash

GTK_THEME=Adwaita:light zenity --info --icon-name checkbox-checked-symbolic --title="This is a test" --width=400 \

--text="""

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Phasellus sed augue at sem varius ornare nec ac est.

Aliquam interdum eget est lobortis vulputate. Pellentesque sollicitudin ultricies risus eu blandit. Nullam feugiat elit dictum ullamcorper tincidunt. Vivamus ut ultricies arcu, non dignissim nulla. Etiam vitae erat vel neque finibus sollicitudin. """

```

# Toggle between themes?

For example: To toggle between dark and light mode

```

#!/usr/bin/env bash

if [ $(gsettings get org.gnome.desktop.interface gtk-theme) == "'Adwaita-dark'" ];

then

gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita'

else

gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita-dark'

fi

```

# Open apps with specific theme

- Show all the installed apps, searching for it’s desktop files in:

  - `/usr/share/applications`

  - `$HOME/.local/share/applications`

  - `/var/lib/flatpak/exports/share/applications/`

  - `/var/lib/snapd/desktop/applications/`

  - other places?

- Edit the line of the desktop file app adding a string after the `Exec=` and `TryExec=` strings in the format bellow

- Create a backup of the file

- If file in `$HOME/.local/share/applications`:

  - save it at the same folder

- If not
  - save a copy in `$HOME/.local/share/applications`

### Format:

Before:

`bash -c '$(GTK_THEME=$theme:$variant `

After:

`)'`

### Example:

The Power Statistic app

Before:

```

Exec=flatpak run org.gnome.PowerStats

```

After:

```

Exec=bash -c '$(GTK_THEME=Adwaita:light flatpak run org.gnome.PowerStats)'

```