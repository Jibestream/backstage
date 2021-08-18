# This dockerfile builds an image for the backend package.
# It should be executed with the root of the repo as docker context.
#
# Before building this image, be sure to have run the following commands in the repo root:
#
# yarn install
# yarn tsc
# yarn build
#
# Once the commands have been run, you can build the image using `yarn build-image`
FROM node:14-buster-slim

WORKDIR /app

ARG phost
ARG pdb
ARG puser
ARG ppass
ARG gh_token
ARG auth_okta_cid
ARG auth_okta_cs
ARG auth_okta_au
ARG auth_gh_cid
ARG auth_gh_cs

ENV POSTGRES_HOST=$phost
ENV POSTGRES_DB=$pdb
ENV POSTGRES_USER=$puser
ENV POSTGRES_PASSWORD=$ppass
ENV GITHUB_TOKEN=$gh_token
ENV AUTH_OKTA_CLIENT_ID=$auth_okta_cid
ENV AUTH_OKTA_CLIENT_SECRET=$auth_okta_cs
ENV AUTH_OKTA_AUDIENCE=$auth_okta_au
ENV AUTH_GITHUB_CLIENT_ID=$auth_gh_cid
ENV AUTH_GITHUB_CLIENT_SECRET=$auth_gh_cs

RUN apt update && apt install -y mkdocs && apt -y install python3-pip
RUN pip3 install mkdocs-techdocs-core

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN yarn install --frozen-lockfile --production --network-timeout 300000 && rm -rf "$(yarn cache dir)"

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY packages/backend/dist/bundle.tar.gz app-config.yaml ./
ADD packages/catalog-model ./catalog-model
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
