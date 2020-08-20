[![Netlify Status](https://api.netlify.com/api/v1/badges/6c35f51e-51c1-493c-8dfe-eb11705061e4/deploy-status)](https://app.netlify.com/sites/cloudytuts/deploys)

# Overview
CloudyTuts is an open source website for news, guides, and tutorials for all things cloud. CloudyTuts uses Hugo to generate a static website, and the contents of the website are stored in this repository.

# Contributions

## As a User
Regular users of CloudyTuts can help us improve the quality of our content and their user experience. As a contributing user your name will be added to the content you contributed to as a **contributor**.

Users can contribute in the following ways:

* Report issues found in our content:
    * Typos
    * Missing instructions
    * Errors in instructions

* Submit corrections to our content:
    * Typos
    * Improved clarity
    * Add additional content

You will need a Github account to contribute as a user. Issues should be logged under the this project's **Issues**.

To submit updates to existing content you will need to fork this repository, make your modifications, and then create a pull request.

## As an Author
We welcome contributions from other authors to help us build our library. 

### Registering as an Author
To be an author on CloudyTuts you must:
* Add your author profile.
* Add a profile image to `/static/images/authors` using Git LFS.
    * Images must be no less than 250x250 pixels.
    * Image files must be less than 20KB.
* Submit at least one news item, guide, or tutorial.

### Author Profile
Authors are defined as YAML files under `/data/_authors`. 

```yaml
name: Shane Rainville
image: serainville.jpg
role: DevOps Engineer / Cloud Specialist
bio: |
  Developer, pipeline builder, cloud engineer, and believer in DevSecOps 
social:
  twitter: shanerainville
  github: serainville
  youtube: shanerainville
  facebook: shanerainville
  website: https://www.cloudytuts.com
```

The required fields for an author are: `name`, `image`, `role`, and `bio`.
Social fields are optional and can be of any kind `twitter`, `github`, `youtube`, `facebook`, and 'website'.

### Submission Reviewal
All author submissions will be reviewed for quality purposes. To be approved, submissions must match the following criteria:
* Well written
* Minimum words
  - 300 words for news
  - 300 words for tutorials
  - 1200 words for guides
* Unique to CloudyTuts (no copy-pasted content from other websites)