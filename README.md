# Pulse Github Action

This is a command-line interface to push events to [Pulse](https://pulse.codacy.com).

Take a look at Pulse's documentation [here](https://docs.pulse.codacy.com).

## Github Action

You can use this Github-Action to send changes and deployment events to Pulse's service
directly from you CI.

The following workflow is an example where [git-version](https://github.com/codacy/git-version) is used to
generate new versions on each deployment and store that information in git tags.

```yaml
name: Pulse

on:
  push:
    branches: ["master"]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
        with:
            # Will fetch all history and tags required to generate version
            fetch-depth: 0

      # Generate previous and next version from git tags
      - name: Git Version
        id: generate-version
        uses: codacy/git-version@2.4.0

      # Push git tag to repository
      - name: "Tag version"
        run: |
          git tag ${{ steps.generate-version.outputs.version }}
          git push --tags "https://codacy:${{ secrets.GITHUB_TOKEN }}@github.com/codacy/pulse-event-cli"

      # ...
      # Deployment steps
      #...

      # Push deployment and changes events to pulse
      - name: "Push data to pulse"
        uses: codacy/pulse-action@0.0.3
        with:
          args: push git deployment \
            --api-key ${{ secrets.PULSE_API_KEY }} \
            --system $GITHUB_REPOSITORY \
            --previous-deployment-ref ${{ steps.generate-version.outputs.previous-version }} \
            --identifier ${{ steps.generate-version.outputs.version }} \
            --timestamp "$(date +%s)"
```
