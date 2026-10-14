# Contributing to Green Thumb

Thanks for helping improve Green Thumb. Documentation fixes, reproducible bug reports, and focused improvements make the project easier to explore and maintain.

## Before making changes

Read the [README](README.md), follow the [setup guide](docs/SETUP.md), and review the [architecture](docs/ARCHITECTURE.md) for the part of the app you want to change. For a substantial feature or architectural change, open an issue first to discuss its scope.

Use your own backend and synthetic test data. Keep API keys, signing files, passwords, account data, and your local Firebase configuration out of issues and pull requests.

## Report a bug or suggest a feature

Use the repository's [issue templates](https://github.com/Yoavsol/GREEN_THUMB/issues/new/choose). For a bug, include the app screen, steps to reproduce, expected and actual behavior, and device/Android version. Redact credentials and account details from logs or screenshots.

For a feature request, describe the user's problem and the behavior you want to improve. A concrete example helps keep the discussion focused.

## Make a pull request

1. Fork the repository and create a branch with a descriptive name, such as `docs/setup-guide` or `fix/calendar-filter`.
2. Keep the change focused and follow the existing Java and XML conventions.
3. Update the documentation if the behavior, configuration, or build requirements change.
4. Validate the affected behavior and describe the results in the pull request.
5. Use a clear title and explain the problem and resulting behavior for a reviewer who has not seen the original discussion.

## Validation

After configuring the project, run the checks relevant to your change:

```bash
./gradlew :app:assembleDebug
./gradlew :app:lintDebug
```

Use `gradlew.bat` on Windows. Test changes to screens, reminders, camera/gallery handling, or network integrations on a device or emulator. The [setup walkthrough](docs/SETUP.md#5-walk-through-the-main-features) provides a starting point.

No automated test sources are currently included, even though test dependencies are declared. If you add tests, explain what behavior they cover. State any validation you could not perform and why.

For documentation-only changes, check local links, headings, code examples, and GitHub's rendered preview. Do not report an Android build or runtime check as passed unless you actually ran it.

## Commit messages

Write a short, specific subject that describes the change. The following format is encouraged:

```text
type(scope): describe the change

Explain the reason for the change and any useful implementation detail.
Include relevant validation or issue references when appropriate.
```

Examples:

- `docs(readme): explain plant-care features and setup`
- `fix(calendar): include tasks due at the end of the selected day`
- `feat(guides): add a title search filter`

Use a body when the reason, tradeoff, or verification would help the reviewer. Keep unrelated changes in separate commits, and avoid vague subjects such as `Finished` or `updates`.

## License

Contributions are made under the project's [MIT License](LICENSE). Preserve applicable copyright notices and the licenses or attribution requirements of third-party material.
