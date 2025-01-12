project('rkdeveloptool', 'cpp', version: '1.3.2', default_options: ['c_std=c2x', 'cpp_std=c++20'])

# make sure we have libusb and acquire it if not
libusb = dependency('libusb-1.0', fallback: ['libusb', 'libusb_dep'])

# set up config file
conf = configuration_data()
conf.set('version', meson.project_version())
configure_file(
    input: 'config.h.in',
    output: 'config.h',
    configuration: conf,
)


# the actual executable
executable(
    'rkdeveloptool',
    'main.cpp',
    'RKBoot.cpp',
    'RKComm.cpp',
    'RKDevice.cpp',
    'RKImage.cpp',
    'RKLog.cpp',
    'RKScan.cpp',
    'crc.cpp',
    dependencies: [libusb],
    install: true,
)


# Install udev rules if udev is available
udev = dependency('udev', required: false)
if udev.found()
    udevdir = udev.get_pkgconfig_variable('udevdir')
    udev_rules_dir = udevdir + '/rules.d'
    install_data(
        ['99-rk-rockusb.rules'],
        install_dir: udev_rules_dir,
    )
else
    message('udev not found, not installing udev rules')
endif


# Build and install the man pages
scdoc = dependency(
    'scdoc',
    native: true,
    required: get_option('man-pages'),
)
if scdoc.found()
    scdoc_prog = find_program(scdoc.get_pkgconfig_variable('scdoc'), native: true)
    sh = find_program('sh', native: true)
    mandir = get_option('mandir')
    man_files = [
        'doc/rkdeveloptool.1.scd',
    ]
    foreach filename : man_files
        topic = filename.split('.')[-3].split('/')[-1]
        section = filename.split('.')[-2]
        output = '@0@.@1@'.format(topic, section)

        custom_target(
            output,
            input: filename,
            output: output,
            command: [sh, '-c', '@0@ < @INPUT@ > @1@'.format(scdoc_prog.path(), output)],
            install: true,
            install_dir: '@0@/man@1@'.format(mandir, section),
        )
    endforeach
else
    message('scdoc not found, will not build docs')
endif
