# Continuous Integration

A CI server is available for the flexiblepower projects on http://fpai-ci.sensorlab.tno.nl/. This server tries to build every commit on every branch for the configured projects (at the time of writing this is powermatcher, fpai-core, fpai-devices and fpai-powermatcher). The builds are placed on http://fpai-ci.sensorlab.tno.nl/builds.

Each commit is built as a snapshot, which is saved for a limited amount of time (they will be cleaned up sometime). These can be used to experiment with the latest development version. For the final release, the build will be put here on GitHub as a real release.