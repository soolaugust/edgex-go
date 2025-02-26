#  ----------------------------------------------------------------------------------
#  Copyright 2019 Dell Technologies, Inc.
#  Copyright 2020 Intel Corp.
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
# 
#  ----------------------------------------------------------------------------------

ARG BUILDER_BASE=golang:1.15-alpine3.12
FROM ${BUILDER_BASE} AS builder

WORKDIR /edgex-go

RUN sed -e 's/dl-cdn[.]alpinelinux.org/nl.alpinelinux.org/g' -i~ /etc/apk/repositories

RUN apk add --update --no-cache make git

COPY go.mod .

RUN go mod download

COPY . .

RUN make cmd/security-file-token-provider/security-file-token-provider \
  cmd/security-secretstore-setup/security-secretstore-setup

FROM alpine:3.12

RUN apk add --update --no-cache ca-certificates dumb-init curl

LABEL license='SPDX-License-Identifier: Apache-2.0' \
      copyright='Copyright (c) 2019: Dell Technologies, Inc.'

WORKDIR /

COPY --from=builder /edgex-go/cmd/security-file-token-provider/res/token-config.json /res-file-token-provider/token-config.json
COPY --from=builder /edgex-go/cmd/security-secretstore-setup/res-file-token-provider/configuration.toml  /res-file-token-provider/configuration.toml
COPY --from=builder /edgex-go/cmd/security-secretstore-setup/res/configuration.toml /res/configuration.toml

COPY --from=builder /edgex-go/cmd/security-file-token-provider/security-file-token-provider .
COPY --from=builder /edgex-go/cmd/security-secretstore-setup/security-secretstore-setup .

# setup the entry point script
COPY --from=builder /edgex-go/cmd/security-secretstore-setup/entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh \
    && ln -s /usr/local/bin/entrypoint.sh /

ENTRYPOINT ["entrypoint.sh"]
