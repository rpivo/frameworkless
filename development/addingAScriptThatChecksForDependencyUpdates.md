## Adding a Script That Checks for Dependency Updates

This npm script uses the [npm-check-updates](https://www.npmjs.com/package/npm-check-updates) package to check if any updates are available for a project's dependencies.

<hr />

`npm x` is an alias for `npm exec`. The `-y` flag suppresses a prompt that asks if the `npm-check-updates` package should be cached locally. The `-u` flag automatically updates the packages that the `npm-check-updates` package finds to update.

```json
"scripts": {
  "update": "npm x -y npm-check-updates@latest -u"
}
```

### Resources

- [NPM / npm-check-updates](https://www.npmjs.com/package/npm-check-updates)
- [NPM / npx](https://docs.npmjs.com/cli/v7/commands/npx)
