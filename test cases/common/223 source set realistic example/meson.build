# a sort-of realistic example that combines the sourceset and kconfig
# modules, inspired by QEMU's build system

project('sourceset-example', 'cpp')
ss = import('sourceset')
kconfig = import('unstable-kconfig')

zlib = dependency('zlib', version : '>=1.2.8', required: false)
not_found = dependency('not-found', required: false)

common = ss.source_set()
specific = ss.source_set()

common.add(files('main.cc'))
common.add(when: zlib, if_true: files('zlib.cc'))
common.add(when: not_found,
           if_true: files('was-found.cc'),
           if_false: files('not-found.cc'))

subdir('boards')
subdir('devices')

if meson.is_unity()
  specific.add_all(common)
  common = ss.source_set()
endif

common_lib = static_library('common', common.all_sources())

targets = [ 'arm', 'aarch64', 'x86' ]
target_dirs = { 'arm' : 'arm', 'aarch64' : 'arm', 'x86': 'x86' }

foreach x : targets
  config = kconfig.load('config' / x)
  target_specific = specific.apply(config, strict: false)
  target_common = common.apply(config, strict: false)
  target_deps = target_specific.dependencies() + target_common.dependencies()
  executable(x,
             objects: common_lib.extract_objects(target_common.sources()),
             sources: target_specific.sources(),
             dependencies: target_deps,
             include_directories: 'boards' / target_dirs[x],
             cpp_args: '-DTHE_TARGET="' + x + '"')
endforeach
