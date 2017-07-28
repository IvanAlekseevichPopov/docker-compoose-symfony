FROM node:6.1

WORKDIR /usr/src/app

# File permissions
ARG PARENT_USER_ID
RUN useradd node; \
    usermod -u $PARENT_USER_ID node

USER node

CMD ["npm run start"]
EXPOSE 3000