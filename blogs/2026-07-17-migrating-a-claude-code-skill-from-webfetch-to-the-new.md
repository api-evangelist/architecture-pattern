---
title: "Migrating a Claude Code skill from WebFetch to the new CircleCI CLI: lessons learned about tool and API design"
url: "http://microservices.io//post/deployment-pipeline/2026/07/17/from-webfetch-to-circleci-cli-lessons-learned.html"
date: "2026-07-17"
feed_url: "https://microservices.io/feed.xml"
---
One of the Claude Code skills in my idea-to-code plugin is debugging-ci-failures . It tells Claude Code how to watch and diagnose a failing CI build. Specifically, it instructs Claude Code to do the following five steps: Watch the build to completion Identify the failing job Fetch the failing step’s output Download the test artifacts Read the JUnit TEST-*.xml files to find the root cause The skill supports two CI systems: GitHub Actions (via the gh CLI) and CircleCI.
