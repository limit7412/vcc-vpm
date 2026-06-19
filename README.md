# VPM Package Listing Template

Starter for making your own Package Listings, including automation for building and publishing them.

Once you're all set up, you'll be able to update the [`source.json`](source.json) file, and generate a listing which works in the VPM for delivering updates for all the listed packages.

## ▶ Getting Started

* Press [![Use This Template](https://user-images.githubusercontent.com/737888/185467681-e5fdb099-d99f-454b-8d9e-0760e5a6e588.png)](https://github.com/vrchat-community/template-package-listing/generate)
to start a new GitHub project based on this template, and follow the directions there. 
  * Choose a fitting repository name and description.
  * Set the visibility to 'Public'. You can also choose 'Private' and change it later.
  * You don't need to select 'Include all branches.'
* Edit this project on GitHub in your web browser, or clone it repository locally using Git.
  * If you're unfamiliar with Git and GitHub, [visit GitHub's documentation](https://docs.github.com/en/get-started/quickstart/).
  
## Setting up the Automation

You'll need to edit some of the files in this template, starting with [`source.json`](source.json):
- Fill out general information about your listing, such as the [`name`](source.json#L2), [`id`](source.json#L3), [`author`](source.json#L5), [`description`](source.json#L10), etc.
- Make sure to update the ["url"](source.json#L4) field on line 4, replacing "vrchat-community" with your GitHub username, and "template-package-listing" with your repo name. This is the link that will be used to download your listing once it's published by GitHub. For example, the user "thupper" who made a repo called "thupper-listing" would update the url to "https://thupper.github.io/thupper-listing/index.json".
- Update the "url" within ["infoLink"](source.json#L11) (on line 11) with the url of this new repo you've created.
- If you'd like to include packages hosted on GitHub, specify them in [`githubRepos`](source.json#L16).
- If you'd like to include packages hosted elsewhere as a `.zip` file, specify them in [`packages`](source.json#L19).
  - You can safely remove either [`githubRepos`](source.json#L16) or [`packages`](source.json#L19) if you're not using them. 
- Finally, go to the "Settings" page for your repo, then choose "Pages", and look for the heading "Build and deployment". Change the "Source" dropdown from "Deploy from a branch" to "GitHub Actions".

## 📃 Rebuilding the Listing

Whenever you make a change to the `main` branch, or when you trigger it manually, the 'Build Repo Listing' action will make a new index of all the releases available and publish them as a website hosted fore free on GitHub Pages. This listing can be used by the VPM to keep your package up to date, and the generated index page can serve as a simple landing page with info for your package. The URL for your package will be in the format https://username.github.io/repo-name.

## 🔄 Auto-Updating on New Releases

The listing is rebuilt automatically when a linked package publishes a new release,
so you don't have to touch `source.json` every time.

### Scheduled detection (no setup required)
The [`detect-release.yml`](.github/workflows/detect-release.yml) workflow runs every
6 hours, checks the latest release of every repository listed in
[`githubRepos`](source.json), and re-triggers the 'Build Repo Listing' action only
when it detects a new release. You can also run it manually from the "Actions" tab
("Detect Package Releases" → "Run workflow"), and you can change the schedule by
editing the `cron` expression in that file.

### Instant updates on release (optional)
If you want the listing to update the moment a package is released (instead of
waiting up to 6 hours), add a workflow like the following to the **package
repository**. It sends a `repository_dispatch` event that immediately triggers the
build here:

```yaml
# .github/workflows/notify-vpm-listing.yml in your package repo
name: Notify VPM Listing
on:
  release:
    types: [published]
jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger listing rebuild
        run: |
          curl -X POST \
            -H "Accept: application/vnd.github+json" \
            -H "Authorization: Bearer ${{ secrets.LISTING_DISPATCH_TOKEN }}" \
            https://api.github.com/repos/limit7412/vcc-vpm/dispatches \
            -d '{"event_type":"package-released"}'
```

`LISTING_DISPATCH_TOKEN` must be a Personal Access Token (or fine-grained token)
with `contents: write` / Actions permission on this listing repository, stored as a
secret in the package repository.

## 🏠 Customizing the Landing Page

The contents of the `Website` directory can be customized to change the appearance of the landing page. Most of the information will be automatically filled in with information from [`source.json`](source.json). Customizing the landing page by hand is not required.

## Technical Stuff

You are welcome to make your own changes to the automation process to make it fit your needs, and you can create Pull Requests if you have some changes you think we should adopt. Here's some more info on the included automation:

### Build Listing
[build-listing.yml](.github/workflows/build-listing.yml)

This is a composite action which builds a vpm-compatible [Repo Listing](https://vcc.docs.vrchat.com/vpm/repos) based on the items you've added to your [`source.json`](source.json) file. you've created. In order to find all your releases and combine them into a listing, it checks out [another repository](https://github.com/vrchat-community/package-list-action) which has a [Nuke](https://nuke.build/) project which includes the VPM core lib to have access to its types and methods. This project will be expanded to include more functionality in the future - for now, the action just calls its `BuildRepoListing` target, which calls `RebuildHomePage` when it completes. If you wanted to make an action that just rebuilds the home page, you could call that directly instead - just copy the existing call and replace the target names.
