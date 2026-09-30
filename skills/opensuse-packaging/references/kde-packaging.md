# KDE Frameworks 6 / Qt6 applications

Owner of: the KF6 build and install macros, the `%{_kf6_*dir}` file-list macros and the
KDE:Extra conventions for ECM-based (Qt6 / Kirigami) packages. The generic CMake rules stay in
`references/specfile-guidelines.md`; this file is where they do not apply.

## KF6 / ECM packages (Qt6, Kirigami)

Applies when upstream's `CMakeLists.txt` has `find_package(ECM … NO_MODULE)` and
`include(KDEInstallDirs)`. Write the KF6 forms from the start: KDE:Extra reviewers decline the
generic ones, and every spec sampled there (haruna, kaidan, koi, drawy, kommit, imprint,
kgeotag, klevernotes) uses the KF6 build macros, the `%{_kf6_*dir}` paths and
`kf6-extra-cmake-modules`.

| generic form | KF6 form |
|---|---|
| `%cmake` + `%cmake_build` | `%cmake_kf6` + `%kf6_build` |
| `%cmake_install` | `%kf6_install` |
| `%{_bindir}` | `%{_kf6_bindir}` |
| `%{_datadir}/applications` | `%{_kf6_applicationsdir}` |
| `%{_datadir}/knotifications6` | `%{_kf6_notificationsdir}` |
| `%{_datadir}/metainfo` | `%{_kf6_appstreamdir}` |
| `%{_datadir}/icons` | `%{_kf6_iconsdir}` |
| `BuildRequires: extra-cmake-modules` | `BuildRequires: kf6-extra-cmake-modules >= X.Y` (X.Y = the ECM version in `find_package(ECM …)`) |

- **Keep `%kf6_build` and `%kf6_install` bare.** spec-cleaner rewrites them to
  `%{kf6_build}` / `%{kf6_install}`; both forms expand the same and build the same, but the
  bare form is what KDE:Extra specs and their reviewers use. Treat that one diff line as an
  accepted spec-cleaner deviation (`references/spec-cleaner.md` "Checking a spec file"), and
  check the rest of the diff is empty.
- **No `gcc-c++` BuildRequires.** None of the sampled specs lists it, and the build finds a C++
  compiler without it; a reviewer flagged it as unneeded.
- **Linking `Qt6::Sql` does not install a database driver.** Grep the source for the name passed
  to `QSqlDatabase::addDatabase` and add the matching `Requires: qt6-sql-<driver>`
  (`QSQLITE` → `qt6-sql-sqlite`). kaidan lists it under both BuildRequires and Requires.
- **Declare the runtime QML imports by hand** when rpm does not detect them, as klevernotes does:
  `kf6-kirigami-imports`, `kirigami-addons6`, `qt6-declarative-imports`, plus any other
  `kf6-*-imports` the QML files import.
- **Translations:** `%lang_package`, `%find_lang %{name} --all-name` in `%install` and
  `%files lang -f %{name}.lang` (7 of the 8 sampled specs).
- **No upstream tests → no `%check` section at all**, not a comment-only placeholder. The
  rpmlint `no-%check-section` warning is expected; a KDE:Extra reviewer asked for exactly that.
- **Sample before writing a new one:** `osc cat KDE:Extra <pkg> <pkg>.spec` for two or three
  current specs (klevernotes is a small Kirigami app). That is the packaging-structure survey of
  Core directive item 9 for this ecosystem.

Real case: transistor (sr#1381075) — declined for the generic `%cmake` macros, `%{_bindir}` /
`%{_datadir}` paths, plain `extra-cmake-modules`, a `gcc-c++` BuildRequires and a missing
SQL-driver `Requires`.
