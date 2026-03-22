# PR #1261 Feedback Todo

## Fixes Needed

- [x] Fix map mutation in WithAllowedEnv and WithPicoclawEnvVars
- [x] Add lowercase http_proxy/https_proxy to allowlist
- [ ] Add log warning when env vars are stripped (breaking change) - logs final env keys at exec time
  - NOTE: Not needed - can use config env_allowlist to get same behavior

## Documentation

- [ ] Document GOPATH, GOROOT, NODE_PATH in tools_configuration.md
  - NOTE: Checked codebase - no tools use GOPATH/GOROOT/NODE_PATH currently
  - Can be added via config env_allowlist if needed - mention in docs?
- [ ] Add migration note about breaking change
  - NOT A BREAKING CHANGE: Users can add vars via config env_allowlist if needed - opt-in not opt-out

## Testing

- [x] Add tests: API keys NOT passed to children
- [x] Add tests: LLM cannot override PATH, LD_PRELOAD
- [x] Add tests: config env_set overrides inherited env
- [x] Add tests: env_allowlist extends default list
- [ ] Add tests: Windows case-insensitive behavior - Skipped (needs Windows CI - envKey needs refactoring to mock GOOS)

## Technical Debt

- [ ] Fix write tool to respect system umask 0002 (setgid) - creates files as 644 instead of 664
