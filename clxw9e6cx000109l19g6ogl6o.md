---
title: "Part 2 - Artifacts Attestation for Builds in Github Actions"
seoTitle: "Attestation Artifact"
seoDescription: "Boost build's supply chain security by adding artifact attestations in Github Actions"
datePublished: Fri Nov 03 2023 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clxw9e6cx000109l19g6ogl6o
slug: part-2-artifacts-attestation-for-builds-in-github-actions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719431048546/3a71e2da-53f9-4e41-8aba-4bd5ff2e33a1.jpeg
tags: deployment, workflow, security, devsecops, actions, pipelines, github-actions-1, attestation

---

Artifact attestations help boost your build's supply chain security by showing where and how your software was created.

In Github Actions we can acheive this using `actions/attest-build-provenance`

1. First add permission to your workflows / re-usable workflows with `attestations: write`
    

```yaml
permissions:
  id-token: write
  contents: read
  attestations: write
```

### (optional) with Maven Builds

need to specify the jar that would be generated during the maven build with exact path. in my case it is `target/*.jar` We can also add the name to attestation.

```yaml
    - name: Build with Maven
      shell: bash
      run: mvn --batch-mode --update-snapshots -DskipTests package    

    - name: Attest Artifacts
      uses: actions/attest-build-provenance@v1.3.2
      with:
        subject-path: 'target/*.jar'
        subject-name: ${{ inputs.container-name }}-attestation
```

2. # with Docker builds
    

Similarly, you can ensure that your Docker artifacts are properly attested, thereby enhancing the security and integrity of your build process. This approach helps in maintaining a robust supply chain security by providing verifiable evidence of how and where your Docker images were created.

```yaml
    - name: Docker build-push
      id: docker-build
      uses: docker/build-push-action@v6
      with:
        context: .
        push: true
        tags: |
          ${{ inputs.acr-url }}/${{ inputs.container-name }}:latest
        # cache-from: type=registry,ref=${{ inputs.acr-url }}/${{ env.containerName }}:buildcache
        # cache-to: type=registry,ref=${{ inputs.acr-url }}/${{ env.containerName }}:buildcache,mode=max
        cache-from: type=gha
        cache-to: type=gha,mode=max   

    - name: Attest Docker build
      uses: actions/attest-build-provenance@v1.3.2
      id: attest
      with:
        subject-name: ${{ inputs.acr-url }}/${{ inputs.container-name }}
        subject-digest: ${{ steps.docker-build.outputs.digest }}
        push-to-registry: true
```

And by just Generating these attestations doesn’t boost security.

3. You need to verify them for it to actually make a difference. Here is an example how this can be verified
    

```yaml
    - name: Verify OCI image
      shell: bash
      env:
        GH_TOKEN: ${{ inputs.github-token }}
      run: |
        gh attestation verify oci://${{ inputs.acr-url }}/${{ inputs.container-name }}:latest --owner "$GITHUB_REPOSITORY_OWNER"
```

You will see a detailed summary after the workflow execution is complete. it will show the status of the Docker build and push actions, indicating whether the image was successfully built and pushed to the specified container registry.

Additionally, it will include information about the attestation process, confirming whether the build provenance was properly attested and pushed to the registry.

Finally, the summary will display the results of the verification step, showing whether the OCI image was successfully verified using the provided GitHub token. This comprehensive summary helps ensure that each step of the workflow was executed correctly and provides valuable insights into the overall process.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719430534520/617391ed-b5ca-49d1-86a9-4047c5e140a2.png align="center")

By clicking on the Attestation URL, you will see detailed information.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719430672824/d084765c-49bb-493e-8cce-76b5e6a0f298.png align="center")