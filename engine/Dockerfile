FROM geoffreybooth/meteor-base:2.0 as bundler

# Install app dependencies
RUN mkdir -p /source
WORKDIR /source
ADD package.json /source/
ADD package-lock.json /source/
RUN NODE_ENV=development meteor npm install

# Build app
ADD . /source/
RUN meteor npm run lint
RUN meteor build --server-only --allow-superuser --directory /



FROM node:12 as runtime

RUN useradd -u 501 -d /home/meteor meteor

# Install app dependencies
COPY --from=bundler /bundle/programs/server/package* /webapp/programs/server/
WORKDIR /webapp/programs/server
RUN NODE_ENV=production npm install

# Copy the app
COPY --from=bundler /bundle /webapp/
RUN NODE_ENV=production npm run install --production
WORKDIR /webapp

HEALTHCHECK --start-period=10s --interval=20s --timeout=1s \
  CMD curl -f http://localhost:3000/graphql -H 'content-type: application/json' --data-binary '{"operationName":null,"variables":{},"query":"{\n  shopInfo {\n    _id\n  }\n}\n"}' || exit

ENV PORT 3000
ENV NODE_ENV production

EXPOSE 3000
USER meteor
CMD node -r ./file_env.js main.js
