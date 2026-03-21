# PR #1261 Feedback - Addressed

## Fixes Applied

1. ✅ Fixed map mutation in WithAllowedEnv and WithPicoclawEnvVars - using maps.Clone
2. ✅ Added lowercase http_proxy/https_proxy/no_proxy to allowlist
3. ✅ Added debug logging of final env keys at exec time
4. ✅ Added comprehensive unit tests for env sanitization

## Not Needed

- **env_sanitize: false option**: Not needed - users can opt-in via config env_allowlist (not breaking)
- **Migration note**: Not needed - this is opt-in via config, not opt-out (not a breaking change)
- **GOPATH/GOROOT/NODE_PATH docs**: Not used in codebase - can be added via config env_allowlist if needed

## Skipped

- **Windows case-insensitive test**: Would need refactoring envKey to mock runtime.GOOS

## Tests Added

- TestWithAllowedEnv_DefaultAllowlist
- TestWithAllowedEnv_ExtraAllowlist
- TestWithAllowedEnv_EnvSetOverride
- TestMergeEnvVars_LLMBlocked
- TestMergeEnvVars_ConfigEnvNotFiltered
- TestMergeEnvVars_PICOCLAWNotOverridable
