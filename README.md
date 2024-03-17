### Publish de um pacote npm usando github actions

Para realizar isso foi demanda de muito tempo então vai valer a pena criar esse README.md kkkkk

1. Criar a pasta .github/workflows
2. Criar um arquivo chamado npm-publish.yml
3. Colar esse código

```bash 
name: Release package
on:
  workflow_dispatch:
    inputs:
      release-type:
        description: 'Release type (one of): patch, minor, major, prepatch, preminor, premajor, prerelease'
        required: true
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      # Checkout project repository
      - name: Checkout
        uses: actions/checkout@v3

      # Setup Node.js environment
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          registry-url: https://registry.npmjs.org/
          node-version: '14'

      # Install dependencies (required by Run tests step)
      #- name: Install dependencies
      #  run: yarn install

      # Tests
      #- name: Run tests
      #  run: yarn test

      # Configure Git
      - name: Git configuration
        run: |
          git config --global user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config --global user.name "GitHub Actions"

      # Bump package version
      # Use tag latest
      - name: Bump release version
        if: startsWith(github.event.inputs.release-type, 'pre') != true
        run: |
          echo "NEW_VERSION=$(npm --no-git-tag-version version $RELEASE_TYPE)" >> $GITHUB_ENV
          echo "RELEASE_TAG=latest" >> $GITHUB_ENV
        env:
          RELEASE_TYPE: ${{ github.event.inputs.release-type }}

      # Bump package pre-release version
      # Use tag beta for pre-release versions
      - name: Bump pre-release version
        if: startsWith(github.event.inputs.release-type, 'pre')
        run: |
          echo "NEW_VERSION=$(npm --no-git-tag-version --preid=beta version $RELEASE_TYPE
          echo "RELEASE_TAG=beta" >> $GITHUB_ENV
        env:
          RELEASE_TYPE: ${{ github.event.inputs.release-type }}

      # Update changelog unreleased section with new version
      - name: Update changelog
        uses: superfaceai/release-changelog-action@v1
        with:
          path-to-changelog: CHANGELOG.md
          version: ${{ env.NEW_VERSION }}
          operation: release

      # Commit changes
      - name: Commit CHANGELOG.md and package.json changes and create tag
        run: |
          git add "package.json"
          git add "CHANGELOG.md"
          git commit -m "chore: release ${{ env.NEW_VERSION }}"
          git tag ${{ env.NEW_VERSION }}

      # Publish version to public repository
      - name: Publish
        run: yarn publish --verbose --access public --tag ${{ env.RELEASE_TAG }}
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

      # Push repository changes
      - name: Push changes to repository
        env:
          GITHUB_TOKEN: ${{ secrets.GIT_TOKEN }}
        run: |
          git push origin && git push --tags

      # Read version changelog
      - id: get-changelog
        name: Get version changelog
        uses: superfaceai/release-changelog-action@v1
        with:
          path-to-changelog: CHANGELOG.md
          version: ${{ env.NEW_VERSION }}
          operation: read

      # Update GitHub release with changelog
      - name: Update GitHub release documentation
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ env.NEW_VERSION }}
          body: ${{ steps.get-changelog.outputs.changelog }}
          prerelease: ${{ startsWith(github.event.inputs.release-type, 'pre') }}
        env:
          GITHUB_TOKEN: ${{ secrets.GIT_TOKEN }}
```
4. Criar os secrets do github para permitir o publish e a troca de versão
5. Criar o CHANGELOG.md
6. Colocar isso aqui no changelog:

```bash
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Changelog
```
Agora toda vez que vocẽ fizer um commit para a main e quiser publicar o seu pacote npm, vocẽ irá em actions (github) e ir em Release package, selecionar run workflow e digitar patch, assim ele começará a rodar o workflow e publicará seu package no npm. Perfeito!
