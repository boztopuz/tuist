# TuistCache

- Only cacheable products are hashed (frameworks, static frameworks, static libraries, dynamic libraries, bundles, macros).
- Test bundles are excluded from binary cache hashing, but test-support frameworks and libraries that link XCTest or Swift Testing can be hashed.
- Explicit cache warm target selection scopes transitive cache candidates from non-test roots only. Focused test roots may remain in the graph but must not expand binary cache candidates.
- Bump the cache version when artifacts become incompatible.
