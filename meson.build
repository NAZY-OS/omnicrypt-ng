project('omnicrypt', version: '0.1.0',
  meson_version: '>= 0.59.0',
  license: 'GPL-3.0-or-later')

python = import('python').find_installation('python3')
gnome = import('gnome')

prefix = get_option('prefix')
bindir = prefix / get_option('bindir')
datadir = prefix / get_option('datadir')
pkgdatadir = datadir / meson.project_name()

subdir('src')
subdir('data')

summary({
  'prefix'   : prefix,
  'bindir'   : bindir,
  'pkgdatadir': pkgdatadir,
})
