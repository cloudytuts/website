# Page Content Types
CloudyTuts posts follows several types of page content.
* Tutorials
* Guides
* News
* Authors

The intention of this guide is to help new contributors submit new content that matches CloudyTuts expectations.

## Tutorials
A tutorial is a post with a limited scope that provides step-by-step instructions. 

To write a new tutorial, create a *Markdown* file in a subdirectory of `/content/en/tutorials` with the following characteristics:
| Page Section |
| ------------ |
| Title        |
| Date         |
| Author       |
| Description  |
| Tags         |

New tutorial posts with the appropriate frontmatter can be dynamically created using a template. In order to create a tutorials post using a Hugo template you can use the following command.

```shell
hugo new tutorials/<section>/<post-title>.md
```

Within the document follow these guidelines:
* Use a maximum of `h2` headings (with two leading `#` characters). The `title` will automatically be wrapped in an `h1` tag, and posts should only contain a single `h1` tag.
* Author must be added. The author should match a filename under `/data/authors/<author-nickname>.yaml`. See authors section below.
* Descriptions will be used for SEO and when content is shared on social media. The description length should be no more than 140 characters.
* Tags help users find related posts. A tag should not, however, have the same name as the section a tutorial is added to. For example, posts under `/content/en/tutorials/kubernetes` should not have a `kubernetes` tag.


## Guides

## Authors
Authors are individuals who submit content to CloudyTuts. In order to add a new author to be associated with a post, you must create a new author data file under `/data/authors`. The filename itself should be a short, unique nickname for the author, which is to be referenced in a posts `author` frontmatter. 

A basic author data file has the following structure:
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
  facebook: cloudytuts
```
