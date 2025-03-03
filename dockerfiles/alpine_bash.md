# Use Alpine Linux (i386) as the base image
ARG TARGETPLATFORM=linux/386
FROM alpine:latest

# Set non-interactive mode
ARG DEBIAN_FRONTEND=noninteractive

# Install required packages including Tmux, Git, Emacs, Zsh, and dependencies
RUN apk update && apk add --no-cache \
    tmux \
    git \
    emacs-nox \
    libxaw \
    libxmu \
    libice \
    libsm \
    libxpm \
    zsh \
    wget \
    curl \
    shadow \
    ncurses \
    fontconfig

# Set a root password
RUN echo "root:root" | chpasswd


# Create a user and set a password
RUN useradd -m user && echo "user:password" | chpasswd

# Fix `su` permission issue
RUN chmod 4755 /bin/su

# Allow `user` to use `sudo` without a password
RUN echo "user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Switch to the user
USER user
WORKDIR /home/user

# Ensure necessary directories exist before writing Powerline fonts
RUN mkdir -p ~/.local/share/fonts ~/.config/fontconfig/conf.d && \
    wget -qO ~/.local/share/fonts/PowerlineSymbols.otf https://github.com/powerline/powerline/raw/develop/font/PowerlineSymbols.otf && \
    wget -qO ~/.config/fontconfig/conf.d/10-powerline-symbols.conf https://github.com/powerline/powerline/raw/develop/font/10-powerline-symbols.conf && \
    fc-cache -vf ~/.local/share/fonts

# Install Oh My Zsh
RUN sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Install Oh My Tmux
RUN git clone https://github.com/gpakosz/.tmux.git ~/.tmux && \
    ln -s -f ~/.tmux/.tmux.conf ~/.tmux.conf && \
    cp ~/.tmux/.tmux.conf.local ~/.tmux.conf.local

# Install Spacemacs
RUN git clone https://github.com/syl20bnr/spacemacs ~/.emacs.d

# Set Zsh as default shell
RUN chsh -s /bin/bash user

# Set working directory for user
WORKDIR /home/user
ENV HOME="/home/user" TERM="xterm" USER="user" SHELL="/bin/bash" EDITOR="emacs" LANG="en_US.UTF-8" LC_ALL="C"
# Set default shell and start as "user"
CMD ["/bin/bash"]
