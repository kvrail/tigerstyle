# TigerStyle for KVRail

[`TIGER_STYLE.md`](TIGER_STYLE.md) is "Adapting TigerStyle to Your Codebase": the coding style
of the [TigerBeetle](https://github.com/tigerbeetle/tigerbeetle) database, restated for other
codebases. It sorts the rules into baseline requirements, project-profile decisions and local
conventions, and adds sections on security, adoption and language differences.

It is an independent adaptation and is not endorsed by the TigerBeetle team. The original is
[TigerStyle](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md).

## Using it in a project

The guide is not meant to be applied whole. A project keeps a short profile file that names the
blocks in force, the mechanisms it chose, every limit with its reaction, and its exceptions
("Adopting the guide" explains the steps). Link the profile to a tagged revision of this
repository, so that a later change to the guide does not change a project's rules silently.

Example: [kvrail/kvrail `docs/style-profile.md`](https://github.com/kvrail/kvrail/blob/main/docs/style-profile.md).

## Revisions

Each revision of the guide is a tag, `r<N>`. See [`CHANGELOG.md`](CHANGELOG.md).

## License

Apache-2.0, the license of TigerStyle itself. See [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).
