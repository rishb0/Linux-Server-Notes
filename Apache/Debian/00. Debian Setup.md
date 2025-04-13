# Debian Setup

This guide outlines the steps to set up a basic Debian system, configure networking, enable SSH, and install essential utilities like Zsh.

### SSH into the Debian System
```
ssh root@192.168.1.25
```

### Check System Architecture
```
dpkg --print-architecture
```

### Update and Upgrade the System
```
apt update
```

```
apt upgrade
```

### Install Basic Tools
```
apt install bash*
```

```
apt install vim net-tools
```

### Install SSH Server and Client
```
apt install openssh-server openssh-client ssh
```

Edit the SSH configuration if needed:
```
vim /etc/ssh/sshd_config
```

Example config change to allow root login:
```
PermitRootLogin yes
```

Enable and start SSH:
```
systemctl enable ssh
```

```
systemctl start ssh
```

```
systemctl restart ssh
```
### Configure Network Interfaces

Edit the interfaces file:
```
vim /etc/network/interfaces
```

Example configuration:
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface (enp0s3)
allow-hotplug enp0s3
iface enp0s3 inet static
    address 192.168.1.25
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8
iface enp0s3 inet6 auto

# The secondary network interface (enp0s8)
allow-hotplug enp0s8
iface enp0s8 inet static
    address 192.168.2.10
    netmask 255.255.255.0
    network 192.168.2.0
    broadcast 192.168.2.255
    dns-nameservers 8.8.8.8
iface enp0s8 inet6 auto
```
### Configure DNS Resolver
```
vim /etc/resolv.conf
```

Example content:
```
nameserver 8.8.8.8
```

### Restart Network Services
```
systemctl restart networking
```

Bring down and up the network interfaces:
```
ifdown enp0s3 && ifup enp0s3
```

```
ifdown enp0s8 && ifup enp0s8
```

Reboot the system:
```
reboot
```
### SSH from a Different User
```
ssh sachin@192.168.1.21
```

Switch to root:
```
su - root
```

### Configure APT Sources

Edit the sources list:
```
vim /etc/apt/sources.list
```

Install transport support:
```
apt install apt-transport-https
```

Example Debian 11 (bullseye) Source List:
```
# deb cdrom:[Debian GNU/Linux 11.6.0 _Bullseye_ - Official amd64 NETINST 20221217-10:42]/ bullseye main
deb http://deb.debian.org/debian/ bullseye main
# deb-src http://deb.debian.org/debian/ bullseye-updates main
```

Example Debian 12 (bookworm) Source List:
```
deb https://deb.debian.org/debian/ bookworm main non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main non-free-firmware
```

### Install Useful Utilities
```
apt install vim fonts-lato fonts-open-sans fonts-roboto fonts-mononoki fonts-indic zsh net-tools curl wget unzip
```

### Reconnect via SSH
```
ssh root@192.168.1.21
```
### Install Zsh and Plugins

Check current shell:
```
echo $SHELL
```

View available shells:
```
cat /etc/shells
```

Install Zsh and plugins:
```
apt install zsh*
```

```
apt install zsh
```

```
apt install zsh-autosuggestions
```

```
apt install zsh-syntax-highlighting
```

Check Zsh version and location:
```
zsh --version
```

```
which zsh
```
### Set Zsh as Default Shell

Set Zsh for current user:
```
chsh -s $(which zsh)
```

Verify:
```
grep zsh /etc/passwd
```

### Configure Zsh

Edit Zsh configuration:
```
nano /root/.zshrc
```

```
# ~/.zshrc file for zsh interactive shells.
# see /usr/share/doc/zsh/examples/zshrc for examples

setopt autocd              # change directory just by typing its name
#setopt correct            # auto correct mistakes
setopt interactivecomments # allow comments in interactive mode
setopt magicequalsubst     # enable filename expansion for arguments of the form ‘anything=expression’
setopt nonomatch           # hide error message if there is no match for the pattern
setopt notify              # report the status of background jobs immediately
setopt numericglobsort     # sort filenames numerically when it makes sense
setopt promptsubst         # enable command substitution in prompt

WORDCHARS=${WORDCHARS//\/} # Don't consider certain characters part of the word

# hide EOL sign ('%')
PROMPT_EOL_MARK=""

# configure key keybindings
bindkey -e                                        # emacs key bindings
bindkey ' ' magic-space                           # do history expansion on space
bindkey '^U' backward-kill-line                   # ctrl + U
bindkey '^[[3;5~' kill-word                       # ctrl + Supr
bindkey '^[[3~' delete-char                       # delete
bindkey '^[[1;5C' forward-word                    # ctrl + ->
bindkey '^[[1;5D' backward-word                   # ctrl + <-
bindkey '^[[5~' beginning-of-buffer-or-history    # page up
bindkey '^[[6~' end-of-buffer-or-history          # page down
bindkey '^[[H' beginning-of-line                  # home
bindkey '^[[F' end-of-line                        # end
bindkey '^[[Z' undo                               # shift + tab undo last action

# enable completion features
autoload -Uz compinit
compinit -d ~/.cache/zcompdump
zstyle ':completion:*:*:*:*:*' menu select
zstyle ':completion:*' auto-description 'specify: %d'
zstyle ':completion:*' completer _expand _complete
zstyle ':completion:*' format 'Completing %d'
zstyle ':completion:*' group-name ''
zstyle ':completion:*' list-colors ''
zstyle ':completion:*' list-prompt %SAt %p: Hit TAB for more, or the character to insert%s
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'
zstyle ':completion:*' rehash true
zstyle ':completion:*' select-prompt %SScrolling active: current selection at %p%s
zstyle ':completion:*' use-compctl false
zstyle ':completion:*' verbose true
zstyle ':completion:*:kill:*' command 'ps -u $USER -o pid,%cpu,tty,cputime,cmd'

# History configurations
HISTFILE=~/.zsh_history
HISTSIZE=1000
SAVEHIST=2000
setopt hist_expire_dups_first # delete duplicates first when HISTFILE size exceeds HISTSIZE
setopt hist_ignore_dups       # ignore duplicated commands history list
setopt hist_ignore_space      # ignore commands that start with space
setopt hist_verify            # show command with history expansion to user before running it
#setopt share_history         # share command history data

# force zsh to show the complete history
alias history="history 0"

# configure `time` format
TIMEFMT=$'\nreal\t%E\nuser\t%U\nsys\t%S\ncpu\t%P'

# make less more friendly for non-text input files, see lesspipe(1)
#[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

configure_prompt() {
    prompt_symbol=㉿
    # Skull emoji for root terminal
    #[ "$EUID" -eq 0 ] && prompt_symbol=💀
    case "$PROMPT_ALTERNATIVE" in
        twoline)
            PROMPT=$'%F{%(#.blue.green)}┌──${debian_chroot:+($debian_chroot)─}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))─}(%B%F{%(#.red.blue)}%n'$prompt_symbol$'%m%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '
            # Right-side prompt with exit codes and background processes
            #RPROMPT=$'%(?.. %? %F{red}%B⨯%b%F{reset})%(1j. %j %F{yellow}%B⚙%b%F{reset}.)'
            ;;
        oneline)
            PROMPT=$'${debian_chroot:+($debian_chroot)}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))}%B%F{%(#.red.blue)}%n@%m%b%F{reset}:%B%F{%(#.blue.green)}%~%b%F{reset}%(#.#.$) '
            RPROMPT=
            ;;
        backtrack)
            PROMPT=$'${debian_chroot:+($debian_chroot)}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))}%B%F{red}%n@%m%b%F{reset}:%B%F{blue}%~%b%F{reset}%(#.#.$) '
            RPROMPT=
            ;;
    esac
    unset prompt_symbol
}

# The following block is surrounded by two delimiters.
# These delimiters must not be modified. Thanks.
# START KALI CONFIG VARIABLES
PROMPT_ALTERNATIVE=twoline
NEWLINE_BEFORE_PROMPT=yes
# STOP KALI CONFIG VARIABLES

if [ "$color_prompt" = yes ]; then
    # override default virtualenv indicator in prompt
    VIRTUAL_ENV_DISABLE_PROMPT=1

    configure_prompt

    # enable syntax-highlighting
    if [ -f /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]; then
        . /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
        ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets pattern)
        ZSH_HIGHLIGHT_STYLES[default]=none
        ZSH_HIGHLIGHT_STYLES[unknown-token]=underline
        ZSH_HIGHLIGHT_STYLES[reserved-word]=fg=cyan,bold
        ZSH_HIGHLIGHT_STYLES[suffix-alias]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[global-alias]=fg=green,bold
        ZSH_HIGHLIGHT_STYLES[precommand]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[commandseparator]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[autodirectory]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[path]=bold
        ZSH_HIGHLIGHT_STYLES[path_pathseparator]=
        ZSH_HIGHLIGHT_STYLES[path_prefix_pathseparator]=
        ZSH_HIGHLIGHT_STYLES[globbing]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[history-expansion]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[command-substitution]=none
        ZSH_HIGHLIGHT_STYLES[command-substitution-delimiter]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[process-substitution]=none
        ZSH_HIGHLIGHT_STYLES[process-substitution-delimiter]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[single-hyphen-option]=fg=green
        ZSH_HIGHLIGHT_STYLES[double-hyphen-option]=fg=green
        ZSH_HIGHLIGHT_STYLES[back-quoted-argument]=none
        ZSH_HIGHLIGHT_STYLES[back-quoted-argument-delimiter]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[single-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[double-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[dollar-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[rc-quote]=fg=magenta
        ZSH_HIGHLIGHT_STYLES[dollar-double-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[back-double-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[back-dollar-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[assign]=none
        ZSH_HIGHLIGHT_STYLES[redirection]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[comment]=fg=black,bold
        ZSH_HIGHLIGHT_STYLES[named-fd]=none
        ZSH_HIGHLIGHT_STYLES[numeric-fd]=none
        ZSH_HIGHLIGHT_STYLES[arg0]=fg=cyan
        ZSH_HIGHLIGHT_STYLES[bracket-error]=fg=red,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-1]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-2]=fg=green,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-3]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-4]=fg=yellow,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-5]=fg=cyan,bold
        ZSH_HIGHLIGHT_STYLES[cursor-matchingbracket]=standout
    fi
else
    PROMPT='${debian_chroot:+($debian_chroot)}%n@%m:%~%(#.#.$) '
fi
unset color_prompt force_color_prompt

toggle_oneline_prompt(){
    if [ "$PROMPT_ALTERNATIVE" = oneline ]; then
        PROMPT_ALTERNATIVE=twoline
    else
        PROMPT_ALTERNATIVE=oneline
    fi
    configure_prompt
    zle reset-prompt
}
zle -N toggle_oneline_prompt
bindkey ^P toggle_oneline_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*|Eterm|aterm|kterm|gnome*|alacritty)
    TERM_TITLE=$'\e]0;${debian_chroot:+($debian_chroot)}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))}%n@%m: %~\a'
    ;;
*)
    ;;
esac

precmd() {
    # Print the previously configured title
    print -Pnr -- "$TERM_TITLE"

    # Print a new line before the prompt, but only if it is not the first line
    if [ "$NEWLINE_BEFORE_PROMPT" = yes ]; then
        if [ -z "$_NEW_LINE_BEFORE_PROMPT" ]; then
            _NEW_LINE_BEFORE_PROMPT=1
        else
            print ""
        fi
    fi
}

# enable color support of ls, less and man, and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    export LS_COLORS="$LS_COLORS:ow=30;44:" # fix ls color for folders with 777 permissions

    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
    alias diff='diff --color=auto'
    alias ip='ip --color=auto'

    export LESS_TERMCAP_mb=$'\E[1;31m'     # begin blink
    export LESS_TERMCAP_md=$'\E[1;36m'     # begin bold
    export LESS_TERMCAP_me=$'\E[0m'        # reset bold/blink
    export LESS_TERMCAP_so=$'\E[01;33m'    # begin reverse video
    export LESS_TERMCAP_se=$'\E[0m'        # reset reverse video
    export LESS_TERMCAP_us=$'\E[1;32m'     # begin underline
    export LESS_TERMCAP_ue=$'\E[0m'        # reset underline

    # Take advantage of $LS_COLORS for completion as well
    zstyle ':completion:*' list-colors "${(s.:.)LS_COLORS}"
    zstyle ':completion:*:*:kill:*:processes' list-colors '=(#b) #([0-9]#)*=0=01;31'
fi

# some more ls aliases
alias ll='ls -l'
alias la='ls -A'
alias l='ls -CF'

# enable auto-suggestions based on the history
if [ -f /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh ]; then
    . /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
    # change suggestion color
    ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=#999'
fi

# enable command-not-found if installed
if [ -f /etc/zsh_command_not_found ]; then
    . /etc/zsh_command_not_found
fi
```
Apply changes:
```
source ~/.zshrc
```

Switch user and set Zsh for another user:
```
su - armour
```

```
chsh -s $(which zsh)
```

```
nano /home/armour/.zshrc
```
# Apache2 Setup on Debian

This guide outlines how to install and configure Apache2, PHP, MySQL, SSL, phpMyAdmin, and WordPress on a Debian-based system.

### System Update

Before beginning, make sure the system is up-to-date.

```
apt update
```

```
apt upgrade
```

### Installing Apache2

Install the Apache web server and basic networking tools:
```
apt install apache2
```

```
apt install net-tools
```

Check running services and confirm Apache installation:
```
netstat -nltup
```

```
dpkg -l | grep apache
```

Edit Apache configuration file:
```
vim /etc/apache2/apache2.conf
```

Restart and enable Apache service:
```
systemctl restart apache2.service
```

```
systemctl enable apache2.service
```

Check network again to ensure Apache is listening:
```
netstat -nltup
```
### Disable Directory Listing

To improve security, disable Apache's directory listing feature:
```
sed -i "s/Options Indexes FollowSymLinks/Options FollowSymLinks/" /etc/apache2/apache2.conf
```

```
systemctl restart apache2.service
```

### Installing PHP

Search for available PHP versions:
```
apt search php | grep "php/stable"
```

Install PHP and commonly used extensions:
```
apt install php php8.2 php8.2-common php8.2-mbstring php8.2-xmlrpc php8.2-soap php8.2-gd php8.2-xml php8.2-intl php8.2-mysql php8.2-cli php8.2-ldap php8.2-zip php8.2-curl php-xml composer
```

Configure PHP settings:
```
vim /etc/php/8.2/apache2/php.ini
```

Example recommended values:
```
memory_limit = 512M
max_execution_time = 500
max_input_vars = 10000
upload_max_filesize = 2048M
post_max_size = 2048M
allow_url_fopen = On
```
Insert the following:
```
<?php
    phpinfo();
?>
```

Set correct ownership:
```
chown -Rv www-data:www-data /var/www/html/phpinfo.php
```
### Installing MySQL Server

Install prerequisites:
```
apt install gnupg
```

```
apt install wget
```

Download MySQL APT configuration package:
```
wget https://dev.mysql.com/get/mysql-apt-config_0.8.24-1_all.deb
```

Install the package:
```
apt install ./mysql-apt-config_0.8.24-1_all.deb
```

Update package lists:
```
apt update
```

Install MySQL server:
```
apt install mysql-community-server
```

Enable and start MySQL:
```
systemctl restart mysql.service
```

```
systemctl enable mysql.service
```
Check open ports:
```
netstat -nltup
```

Run secure installation:
```
mysql_secure_installation
```

Access MySQL:
```
mysql -u root -p
```

Inside MySQL:
```
show databases;
```

Create remote root user (optional for remote access):
```
CREATE USER 'root'@'%' IDENTIFIED WITH caching_sha2_password BY '***';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```
### Enabling SSL on Apache

Enable SSL module and default SSL site:
```
a2enmod ssl
```

```
systemctl restart apache2
```

```
a2ensite default-ssl
```

```
systemctl reload apache2
```

Check network again:
```
netstat -nltup
```

```
service apache2 reload
```
### phpMyAdmin Installation

Download and extract phpMyAdmin:
```
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.0/phpMyAdmin-5.2.0-all-languages.zip
```

```
unzip phpMyAdmin-5.2.0-all-languages.zip
```

Move to web directory:
```
mv -v phpMyAdmin-5.2.0-all-languages /var/www/html/phpmyadmin
```

```
cd /var/www/html
```

Set ownership:
```
chown -Rv www-data:www-data /var/www/html/phpmyadmin
```

Create config file:
```
cp -v /var/www/html/phpmyadmin/config.sample.inc.php /var/www/html/phpmyadmin/config.inc.php
```

```
chown -Rv www-data:www-data config.inc.php
```

Generate a blowfish secret:
```
pwgen 32 -1
```
Edit the config file:
```
vim /var/www/html/phpmyadmin/config.inc.php
```

Insert:
```
$cfg['blowfish_secret'] = 'ophixah6ufboshae9veipahK4daeqah';
```

MySQL remote login example:
```
mysql -h 192.168.1.23 -u root -p
```
### Creating a Self-Signed SSL Certificate
```
mkdir /etc/apache2/ssl
```

Generate a certificate:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/armour.local.key -out /etc/apache2/ssl/armour.local.crt
```

View certificates:
```
cat /etc/apache2/ssl/armour.local.key
```

```
cat /etc/apache2/ssl/armour.local.crt
```

Update hosts file:
```
vim /etc/hosts
```

Example:
```
192.168.1.45    armour.local www.armour.local
```

### Configure Apache Virtual Host for HTTPS

Create the site directory:
```
mkdir -p /var/www/html/armour.local
```

Create the SSL config:
```
vim /etc/apache2/sites-available/armour.local-ssl.conf
```

Example config:
```
<VirtualHost *:443>
    ServerAdmin webmaster@armour.local
    ServerName armour.local
    ServerAlias www.armour.local
    DocumentRoot /var/www/html/armour.local
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/armour.local.crt
    SSLCertificateKeyFile /etc/apache2/ssl/armour.local.key
    <Directory /var/www/html/armour.local>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/armour.local-error.log
    CustomLog ${APACHE_LOG_DIR}/armour.local-access.log combined
</VirtualHost>
```

Enable and restart:
```
a2ensite armour.local-ssl.conf
```
### Install WordPress

Download and extract:
```
wget https://wordpress.org/latest.zip
```

```
unzip latest.zip
```

Move to appropriate directory:
```
mv -v wordpress/* /var/www/html/armour.local
```

Set permissions:
```
chown -Rv www-data:www-data /var/www/html/armour.local
```

### Virtual Host Binding and Multi-site Setup

Create directories:
```
mkdir /var/www/html/site1
mkdir /var/www/html/site2
mkdir /var/www/html/site3
```

Set ownership:
```
chown -R www-data:www-data /var/www/html/site1/
chown -Rv www-data:www-data /var/www/html/site*
```

Configure each site's .conf files (site1, site2, site3):
```
<VirtualHost 192.168.1.25:80>
    ServerName armour.local
    DocumentRoot /var/www/html/site1/
    <Directory /var/www/html/site1/>
        Options FollowSymLinks
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
    ErrorLog /var/log/apache2/site1-error.log
    CustomLog /var/log/apache2/site1-access.log common
    ServerAlias www.armour.local
</VirtualHost>
```

Enable sites and modules:
```
a2dissite 000-default
a2dissite default-ssl
a2ensite site1.conf
a2enmod rewrite
systemctl restart apache2
```

List enabled/available sites:
```
ls -lh /etc/apache2/sites-enabled
ls -lh /etc/apache2/sites-available
```
 
