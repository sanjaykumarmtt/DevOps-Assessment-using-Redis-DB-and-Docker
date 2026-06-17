# Dockerfile for building a custom image based on Ubuntu 22.04
FROM ubuntu:22.04

# Set the environment variable to avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install necessary packages: OpenSSH server, Python 3, and sudo
RUN apt-get update && apt-get install -y openssh-server python3 sudo

# Create a new folder for the SSH daemon to run
RUN mkdir /var/run/sshd

# Set the root password to 'root123' (you can change this to a more secure password)
# RUN echo 'root:root123' | chpasswd

# Configure SSH to allow root login by modifying the sshd_config file
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

RUN mkdir -p /root/.ssh && chmod 700 /root/.ssh

COPY authorized_keys /root/.ssh/authorized_keys

RUN chmod 600 /root/.ssh/authorized_keys

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]