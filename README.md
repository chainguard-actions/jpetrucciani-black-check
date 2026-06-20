# black-check

GitHub Action for [black](https://github.com/psf/black)

## Inputs

### `path`

**Optional** The path to run black on

**Default** `"."`

### `black_flags`

**Optional** Optional black flags (refer to `black --help`)

**Default** `""`

## Outputs

None

## Example usage

```yaml
uses: jpetrucciani/black-check@master

# or specify a path!
uses: jpetrucciani/black-check@master
with:
  path: '.'

# or specify more flags!
uses: jpetrucciani/black-check@master
with:
  black_flags: '--exclude ./env/'
```

## Privacy

This Action contacts Chainguard's licensing server to verify authorization. Connection metadata (IP address, GitHub repository identifier, timestamp, and any metadata encoded in the auth token) is transmitted to Chainguard, Inc. even if authorization is denied in accordance with our [Privacy Notice](https://www.chainguard.dev/legal/privacy-notice)
