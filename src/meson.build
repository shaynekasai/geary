# Version

if meson.project_version().endswith('-dev')
  # Development build
  geary_version_vala = vcs_tag(
    command: '../build-aux/git_version.py',
    input: 'geary-version.vala.in',
    output: 'geary-version.vala',
    replace_string: '@VERSION@'
  )
else
  # Release build
  geary_version_vala = configure_file(
    input: 'geary-version.vala.in',
    output: 'geary-version.vala',
    configuration: conf,
  )
endif

# Common vala options
geary_vala_options = [
  '--target-glib=@0@'.format(target_glib),
  '--thread',
  '--enable-checking',
]

# Symbols for valac's preprocessor must be defined as compiler args,
# not in the code or in config.h
if reference_tracking
  geary_vala_options += [ '--define=REF_TRACKING' ]
endif
if not poodle
  geary_vala_options += [ '--define=DISABLE_POODLE' ]
endif

geary_c_options = [
  '-include', 'config.h',
  # Select libunwind's optimised, local-only backtrace unwiding. See
  # libunwind(3).
  '-DUNW_LOCAL_ONLY',
  # Neither GOA nor GCK want to hang out unless you are cool enough
  '-DGOA_API_IS_SUBJECT_TO_CHANGE',
  '-DGCK_API_SUBJECT_TO_CHANGE',
]

subdir('sqlite3-unicodesn')
subdir('engine')
subdir('client')
subdir('console')
subdir('mailer')

# Web process extension library
geary_web_process = library('geary-web-process',
  join_paths('client', 'web-process', 'web-process-extension.vala'),
  c_args: geary_c_options,
  dependencies: [
    gee,
    gmime,
    webkit2gtk_web_extension ,
    geary_engine_dep,
  ],
  install: true,
  install_dir: web_extensions_dir
)

# Now finally, make the geary executable
geary_bin_sources = files(
  join_paths('client', 'application', 'main.vala'),
)

geary_bin_sources += [
  geary_compiled_schema,
  geary_resources # Included here so they show up in the executable.
]
geary_bin_dependencies = [
  gdk,
  geary_client_dep,
  geary_engine_dep,
  gee,
  gmime,
  goa,
  gtk,
  libmath,
  libsoup,
  webkit2gtk,
]

geary_bin = executable('geary',
  geary_bin_sources,
  dependencies: geary_bin_dependencies,
  vala_args: geary_vala_options,
  c_args: geary_c_options,
  install: true,
)

if enable_valadoc
  geary_docs = custom_target('valadoc',
    build_by_default: true,
    depends: geary_engine_lib,
    input: geary_engine_sources,
    output: 'valadoc',
    command: [ valadoc,
      '--verbose',
      '--force',
      '--internal',
      '--package-name=@0@-@1@'.format(meson.project_name(), meson.project_version()),
      '--package-version=@0@'.format(meson.project_version()),
      '--target-glib=@0@'.format(target_glib),
      '-b', meson.current_source_dir(),
      '-o', '@OUTPUT@',
      '--vapidir=@0@'.format(vapi_dir),
      '--vapidir=@0@'.format(meson.current_build_dir()),
      # Hopefully, Meson gets baked-in valadoc support, so we don't have to do this (see also https://github.com/mesonbuild/meson/issues/894)
      '--pkg', 'glib-2.0', '--pkg', 'gio-2.0', '--pkg', 'gee-0.8', '--pkg', 'sqlite3', '--pkg', 'gmime-2.6', '--pkg', 'javascriptcoregtk-4.0', '--pkg', 'libxml-2.0', '--pkg', 'libunwind',
      '@INPUT@',
    ]
  )
endif
