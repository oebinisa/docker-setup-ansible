FROM ubuntu:latest

# Install OpenSSH server
# RUN apt-get update && apt-get install -y openssh-server
RUN apt-get install -y openssh-server

# Create the SSH directory and generate SSH host keys
RUN mkdir /var/run/sshd && ssh-keygen -A

# Expose SSH port
EXPOSE 22

# Start SSH daemon
CMD ["/usr/sbin/sshd", "-D"]
