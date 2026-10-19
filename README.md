# QMinh: libsndfile 1.0.28 với AddressSanitizer

## Source

- Release: `libsndfile 1.0.28`
- Archive: `libsndfile-1.0.28-release.tar.gz`
- SHA-256: `1ff33929f042fa333aed1e8923aa628c3ee9e1eb85512686c55092d1e5a9dfa9`

## Build

```bash
cd libsndfile-1.0.28
TMPDIR="$PWD/tmp" ./configure \
  --prefix="$PWD/install-asan" \
  --disable-shared --enable-static \
  CC=gcc \
  CFLAGS="-O1 -g -fsanitize=address -fno-omit-frame-pointer" \
  LDFLAGS="-fsanitize=address"
make -j2
ASAN_OPTIONS=detect_leaks=0:abort_on_error=1 \
  LSAN_OPTIONS=detect_leaks=0 make check -j2
make install
```

The environment did not have FLAC/Ogg/Vorbis development libraries, so this
build includes WAV and the built-in formats but does not enable those external
codecs. The installed static library and ASan-enabled command-line programs
are under `libsndfile-1.0.28/install-asan/`.

## Verification

```bash
ASAN_OPTIONS=detect_leaks=0:abort_on_error=1 \
  ./programs/sndfile-info ../file_example_WAV_2MG.wav
```

The sample was parsed successfully. `ldd programs/sndfile-info` confirms that
the executable loads `libasan.so.8`. The test log is saved as
`libsndfile-1.0.28/qminh-make-check.log`; all enabled format tests passed. Leak
detection is disabled for the suite because LeakSanitizer is not stable in this
WSL execution environment; ASan instrumentation remains enabled. The log notes
that FLAC/Ogg/Vorbis tests were skipped because those optional libraries were
not enabled. The sample run is saved as `qminh-sample-asan.log`.

## Git handoff

Commit the release source, the release archive, the WAV seed, this README and
`.gitignore`. Build output, object files, generated test data, ASan install
files and verification logs are ignored.

```bash
git add .gitignore README.md \
  libsndfile-1.0.28-release.tar.gz \
  file_example_WAV_2MG.wav \
  libsndfile-1.0.28/
git status --short
git diff --cached --stat
git commit -m "Add libsndfile 1.0.28 ASan build setup"
git push -u origin <your-branch>
```
