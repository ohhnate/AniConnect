# .github/ISSUE_TEMPLATE/bug_report.md

name: Bug Report
description: File a bug report
title: "[Bug]: "
labels: ["bug", "triage"]
body:

- type: markdown
    attributes:
      value: |
        Thanks for taking the time to fill out this bug report!
- type: textarea
    id: what-happened
    attributes:
      label: What happened?
      description: Also tell us what you expected to happen
    validations:
      required: true
- type: dropdown
    id: browsers
    attributes:
      label: What browsers are you seeing the problem on?
      multiple: true
      options:
        - Firefox
        - Chrome
        - Safari
        - Microsoft Edge
