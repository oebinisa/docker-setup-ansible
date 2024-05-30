FROM alpine:latest

# Install openssh and openrc
RUN apk update && apk add --no-cache openssh openrc

# Set up OpenRC
RUN rc-update add sshd && mkdir -p /run/openrc && touch /run/openrc/softlevel

# Generate SSH host keys
RUN ssh-keygen -A  

# Expose SSH port            
EXPOSE 22       

# Start SSH daemon               
CMD ["/usr/sbin/sshd", "-D"]   