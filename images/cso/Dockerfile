# Copyright 2019 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM --platform=${BUILDPLATFORM} docker.io/alpine/helm:3.12.2 as helm

# Build the manager binary
FROM --platform=${BUILDPLATFORM} docker.io/library/golang:1.20.4@sha256:690e4135bf2a4571a572bfd5ddfa806b1cb9c3dea0446ebadaf32bc2ea09d4f9 as build
ARG TARGETOS TARGETARCH

COPY . /src/cluster-stack-operator
WORKDIR /src/cluster-stack-operator
RUN --mount=type=cache,target=/root/.cache --mount=type=cache,target=/go/pkg \
    GOOS=${TARGETOS} GOARCH=${TARGETARCH} CGO_ENABLED=0 \
    go build -mod=vendor -ldflags "$(hack/version.sh) -extldflags '-static'" \
    -o manager cmd/main.go

FROM --platform=${BUILDPLATFORM} docker.io/library/alpine:3.18.0
WORKDIR /
COPY --from=build /src/cluster-stack-operator/manager .
COPY --from=helm --chown=root:root --chmod=755 /usr/bin/helm /usr/local/bin/helm

# Use uid of nonroot user (65532) because kubernetes expects numeric user when applying pod security policies
USER 65532:65532
ENTRYPOINT ["/manager"]
