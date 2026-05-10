# shellcheck disable=SC2034,SC2148,SC2154
# Maintainer: Toria <ninetailedtori@uwu.gal>
# Maintainer: Kodehawa <david.alejandro.rubio at gmail.com>
# Contributor: FivePB <admin@fivepb.me>
# Contributor: Auxim <hello@auxim.dev>

# Original discord_arch_electron PKGBUILD:
# Maintained by johnnyapol (arch@johnnyapol.me)

# Original maintainers below:
# Based off the discord community repo PKGBUILD by Filipe Laíns (FFY00) <lains@archlinux.org>
# Maintainer: Anna <morganamilo@gmail.com>
# Maintainer: E5ten <e5ten.arch@gmail.com>
# Maintainer: Parker Reed <parker.l.reed@gmail.com>
# Maintainer: Stephanie Wilde-Hobbs <steph@rx14.co.uk>
# Contributor: Cayde Dixon <me@cazzar.net>
# Contributor: Anthony Anderson <aantony4122@gmail.com>

# Color codes
readonly _RED='\033[0;31m'
readonly _RESET='\033[0m'

# print error :3
error() {
    printf '%b[ERROR]%b %s\n' "${_RED}" "${_RESET}" "$1" >&2
    exit 1
}

_pkgname='discord-canary'
_pkgver=$(curl -sI https://discordapp.com/api/download/canary?platform=linux | \
    grep -i '^location:' | sed -E 's|.*apps/linux/([0-9.]+)/.*|\1|')

if [ -z "${_pkgver}" ]; then
    error 'Failed to fetch Discord Canary version from location header'
fi

pkgname="${_pkgname}-electron-bin"
pkgver="${_pkgver}"
pkgrel=1
pkgdesc='Discord Canary (popular voice + video app) using the system provided electron for increased security and performance'
arch=('x86_64')
provides=("${_pkgname}")
conflicts=("${_pkgname}")
url='https://canary.discordapp.com'
license=('custom')
options=(!strip)
install='discord-canary-electron-bin.install'
depends=(
    'alsa-lib'
    'electron35'
    'glibc'
    'gtk3'
    'libcups'
    'libnotify'
    'libxss'
    'nspr'
    'nss'
    'xdg-utils'
)
makedepends=('asar' 'curl' 'sed')
optdepends=(
    'libpulse: Pulseaudio support'
    'xdg-utils: Open files'
    'noto-fonts-emoji: Google font for emoji support.'
    'ttf-symbola: Font for emoji support.'
    'noto-fonts-cjk: Font for special characters such as /shrug face.'
)

source=(
    "${_pkgname}-${pkgver}.tar.gz::https://dl-canary.discordapp.net/apps/linux/${pkgver}/${_pkgname}-${pkgver}.tar.gz"
    'LICENSE.html::https://discordapp.com/terms'
    'OSS-LICENSES.html::https://discordapp.com/licenses'
)
sha256sums=('a7ea26f058f3d5aac89cb432e0300c06b0f533a98513c724c929eb17d348e96d'
            '16936e2f9f2df4f063db742188492f620ef9a541cd06c3c261333dcb20812fbd'
            '24a62492494c1a129940af395d3af054b9faf51706be115ab03db6c911fa5ea9')

prepare() {
    # Path variables
    export _unpackdir="${srcdir}/DiscordCanary"
    export _fakehomedir="${srcdir}/fakehome"
    export _configdir=".config/discordcanary"
    export _appdir="${_configdir}/app-${pkgver}"
    export _libdir="/usr/lib/${_pkgname}"
    export _sharedir="/usr/share/${_pkgname}"

    mkdir -p "${_fakehomedir}/${_configdir}"

    # bootstrap in a subshell so we don't need to cd back out
    (
        cd "${_unpackdir}" || return 1

        HOME="${_fakehomedir}" ./updater_bootstrap --no-zenity \
            "${_fakehomedir}/${_configdir}" canary \
            'https://updates.discord.com/' || {
                error 'bootstrap process failed'
            }

        sleep 2

        cp "${_fakehomedir}/${_appdir}/resources/app.asar" "${srcdir}/" || {
            error 'bootstrap didnt find app.asar'
        }
    ) || exit 1

    # re-extract clean tarball for packaging
    tar xf "${_pkgname}-${pkgver}.tar.gz"
    cd "${_unpackdir}" || exit 1

    # update launcher to point to our binary
    sed -i "s|Exec=.*|Exec=/usr/bin/${_pkgname}|" \
        "${_pkgname}.desktop"
    echo 'Path=/usr/bin' >> "${_pkgname}.desktop"
}

package() {
    cd "${srcdir}" || exit 1

    echo 'Patching app.asar...'

    # unpack app.asar so we can patch it for system electron
    asar e "app.asar" "app"

    # tell discord to look in /usr/lib instead of bundled resources
    grep -q "resourcesPath=path_1.default" "app/bundle.js" || error 'Pattern not found. Check app/bundle.js or update sed pattern.'
sed -i 's/resourcesPath=path_1.default\.join(require\.main\.filename,"\.\.","\.\."),/resourcesPath=process.resourcesPath,/g' "app/bundle.js"
grep -q "resourcesPath=process.resourcesPath" "app/bundle.js" || error 'Replacement failed.'

    echo 'Patch complete. Repacking and completing install.'

    # repack the patched app
    asar p "app" "app.asar" --unpack-dir '**'
    rm -rf "app"

    # create package directories
    install -d \
        "${pkgdir}${_libdir}" \
        "${pkgdir}/usr/share/pixmaps" \
        "${pkgdir}/usr/share/applications" \
        "${pkgdir}/usr/bin"

    # install app
    install -Dm 644 "app.asar"                          "${pkgdir}${_libdir}/app.asar"

    # desktop entry and icon
    install -Dm 644 "${_unpackdir}/discord.png"         "${pkgdir}/usr/share/pixmaps/${_pkgname}.png"
    install -Dm 644 "${_unpackdir}/${_pkgname}.desktop" "${pkgdir}/usr/share/applications/${_pkgname}.desktop"

    # create launcher script that calls system electron
    cat > "${_pkgname}" << 'EOF'
#!/bin/sh
exec electron35 /usr/lib/discord-canary/app.asar "$@"
EOF

    # install launcher script :3
    install -Dm 755 "${_pkgname}"                       "${pkgdir}/usr/bin/${_pkgname}"

    # licenses
    install -Dm 644 'LICENSE.html'                      "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE.html"
    install -Dm 644 'OSS-LICENSES.html'                 "${pkgdir}/usr/share/licenses/${pkgname}/OSS-LICENSES.html"
}
