VERSION 0.8

PROJECT blue-build/earthly-lib

all:
	WAIT
		BUILD ./cargo+build
		BUILD ./titanoboa+build
	END

cosign:
  FROM ghcr.io/sigstore/cosign/cosign:v3.1.1
  SAVE ARTIFACT /ko-app/cosign

sign:
  FROM alpine
  ARG EARTHLY_PUSH

  IF [ "$EARTHLY_PUSH" = "true" ]
	  COPY +cosign/cosign /usr/bin/
	  COPY cosign.pub /

		ARG --required IMAGE
	  COPY --pass-args +digest/digest /
	  LET image="$(echo "$IMAGE" | awk -F ':' '{print $1}')@$(cat /digest)"

	  ENV COSIGN_YES="true"
	  RUN --no-cache \
	    --secret GH_TOKEN \
	    --secret GH_ACTOR \
		  --secret COSIGN_PRIVATE_KEY \
		  --secret COSIGN_PASSWORD \
	    echo "$GH_TOKEN" | cosign login ghcr.io --username "$GH_ACTOR" --password-stdin \
	    && cosign sign \
	      --new-bundle-format=false \
	      --use-signing-config=false \
	      --key=env://COSIGN_PRIVATE_KEY \
	      --recursive \
	      "${image}" \
			&& cosign verify --key=/cosign.pub "${image}"
  END

digest:
  FROM alpine
  RUN apk update && apk add skopeo jq

  ARG --required IMAGE
  RUN skopeo inspect "docker://$IMAGE" | jq -r '.Digest' > /digest
  SAVE ARTIFACT /digest
