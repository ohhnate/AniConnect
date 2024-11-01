# .github/ISSUE_TEMPLATE/feature_request.md

name: Feature Request
description: Suggest an idea for AniConnect
title: "[Feature]: "
labels: ["enhancement"]
assignees: ""

body:

- type: markdown
    attributes:
      value: |
        Thanks for taking the time to suggest a new feature! Please fill out this form as completely as possible.

- type: textarea
    id: problem
    attributes:
      label: Is your feature request related to a problem?
      description: A clear description of what the problem is. Ex. I'm always frustrated when [...]
    validations:
      required: true

- type: textarea
    id: solution
    attributes:
      label: Describe the solution you'd like
      description: A clear description of what you want to happen.
    validations:
      required: true

- type: textarea
    id: alternatives
    attributes:
      label: Describe alternatives you've considered
      description: A clear description of any alternative solutions or features you've considered.

- type: dropdown
    id: scope
    attributes:
      label: Feature Scope
      description: What area of the application does this feature affect?
      multiple: true
      options:
        - Frontend
        - Backend
        - Database
        - Authentication
        - Media Integration
        - Social Features
        - User Experience
        - Performance
        - Security

- type: textarea
    id: context
    attributes:
      label: Additional context
      description: Add any other context about the feature request here.
