project('copyright-autoheader', 'c',
        version: '0.0.1',
        license: 'apache2',
        meson_version: '>= 0.49',
)

install_data(
    files('src/copyright-autoheader'),
    install_dir : get_option('bindir'),
)

# meson modules
python = import('python').find_installation('python3', required : true)

python3_required_modules = ['yaml']

# meson's python.dependency() is not working yet
# use this code picked from gtk-doc as a workaround
foreach p : python3_required_modules
    # Source: https://docs.python.org/3/library/importlib.html#checking-if-a-module-can-be-imported
    script = 'import importlib.util; import sys; exit(1) if importlib.util.find_spec(\''+ p +'\') is None else exit(0)'
    if run_command(python, '-c', script).returncode() != 0
        error('Required Python3 module \'' + p + '\' not found')
    endif
endforeach

rst2man = find_program(['rst2man', 'rst2man3'], required : true)
custom_target('manpage',
    output : 'copyright-autoheader.1',
    input : 'doc/copyright-autoheader.rst',
    command : [rst2man, '@INPUT@', '@OUTPUT@'],
    build_by_default : true,
    install : true,
    install_dir : get_option('mandir') / 'man1',
)

meson.add_dist_script('src/copyright-autoheader')
