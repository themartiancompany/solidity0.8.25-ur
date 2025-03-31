# SPDX-License-Identifier: AGPL-3.0

#    ----------------------------------------------------------------------
#    Copyright © 2024, 2025  Pellegrino Prevete
#
#    All rights reserved
#    ----------------------------------------------------------------------
#
#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU Affero General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU Affero General Public License for more details.
#
#    You should have received a copy of the GNU Affero General Public License
#    along with this program.  If not, see <https://www.gnu.org/licenses/>.

# Maintainer: Truocolo <truocolo@aol.com>
# Maintainer: Truocolo <truocolo@0x6E5163fC4BFc1511Dbe06bB605cc14a3e462332b>
# Maintainer: Pellegrino Prevete (dvorak) <pellegrinoprevete@gmail.com>
# Maintainer: Pellegrino Prevete (dvorak) <dvorak@0x87003Bd6C074C713783df04f36517451fF34CBEf>
# Contributor: Matheus <matheusgwdl@protonmail.com>
# Contributor: Serge K <arch@phnx47.net>
# Contributor: Nicola Squartini <tensor5@gmail.com>

# shellcheck disable=SC2034
# shellcheck disable=SC2154
_os="$( \
  uname \
    -o)"
_pkg="solidity"
pkgver="0.8.26"
pkgname="${_pkg}${pkgver}"
pkgrel="1"
pkgdesc="Smart contract programming language."
arch=(
  "x86_64"
  "i686"
  "aarch64"
  "arm"
  "armv7l"
  "armv6l"
  "mips"
  "powerpc"
  "pentium4"
)
_http="https://github.com"
_ns="ethereum"
url="${_http}/${_ns}/${_pkg}"
license=(
  "GPL-3.0-or-later"
)
depends=()
if [[ "${_os}" == "Android" ]]; then
  depends+=(
    "boost"
  )
elif [[ "${_os}" == "GNU/Linux" ]]; then
  depends+=(
    "boost-libs"
  )
fi
optdepends=(
  "cvc4: SMT checker"
  "z3: SMT checker"
)
makedepends=(
  "cmake"
)
if [[ "${_os}" == "Android" ]]; then
  makedepends+=(
    "boost-headers"
    "boost-static"
  )
elif [[ "${_os}" == "GNU/Linux" ]]; then
  makedepends+=(
    "boost"
  )
fi
checkdepends=(
  "evmone"
)
provides=(
  "solidity=${pkgver}"
  "solc=${pkgver}"
)
conflicts=(
  "${_pkg}-bin"
  "${_pkg}-git"
)
source=(
  "${_pkg}-v${pkgver}.tar.gz::${url}/releases/download/v${pkgver}/${_pkg}_${pkgver}.tar.gz"
)
sha512sums=(
  'ac799b64a792a857469e06d258225865acfca715b191d0e8df471429827725e0cbe631e57938f2bb7560e3d1629223ef0e24652480f385ed581ac96269fe2e29'
)

_boost_version_get() {
  pacman \
    -Qi \
    boost | \
    grep \
      "Version" | \
      awk \
        '{print $3}' | \
        rev | \
          cut \
            -d \
              "-" \
            -f \
	      2 | \
            rev
}

_verlte() {
  printf \
    '%s\n' \
    "${1}" \
    "${2}" | \
    sort \
      -C \
      -V
}

_verlt() {
  ! _verlte \
      "${2}" \
      "${1}"
}

_compile() {
  local \
    _tests="${1}" \
    _boost_version \
    _cmake_opts=() \
    _cxxflags=()
  _cxxflags=(
    "${CXXFLAGS}"
  )
  if [[ "${_os}" == "Android" ]]; then
    _cxxflags+=(
      -Wno-unused-but-set-variable
    )
    _boost_version="$( \
      _boost_version_get)"
    if _verlt \
	   "${_boost_version}" \
	   "1.86.0"; then
      echo \
        "Installed boost version" \
	"<=1.86.0, building with" \
        "deprecated declarations." \
        "Also be sure to use Clang" \
	"<19.x"
      _cxxflags+=(
        -Wno-deprecated-declarations
      )
    fi
  fi
  _cmake_opts=(
    -D CMAKE_BUILD_TYPE="None"
    -D CMAKE_INSTALL_PREFIX="/usr/"
    -D CMAKE_EXECUTABLE_SUFFIX="${pkgver}"
    -D ONLY_BUILD_SOLIDITY_LIBRARIES="OFF"
    -D PEDANTIC="ON"
    -D PROFILE_OPTIMIZER_STEPS="OFF"
    -D SOLC_LINK_STATIC="OFF"
    -D SOLC_STATIC_STDLIBS="OFF"
    -D STRICT_NLOHMANN_JSON_VERSION="OFF"
    -D STRICT_Z3_VERSION="OFF"
    -D TESTS="${_tests}"
    -D USE_LD_GOLD="OFF"
    -D USE_SYSTEM_LIBRARIES="OFF"
    -S "${srcdir}/${_pkg}_${pkgver}/"
    -Wno-dev
  )
  export \
    CXXFLAGS="${_cxxflags[*]}"
  CXXFLAGS="${_cxxflags[*]}" \
  cmake \
    -B \
      "${srcdir}/${_pkg}_${pkgver}/build/" \
    "${_cmake_opts[@]}"
  CXXFLAGS="${_cxxflags[*]}" \
  cmake \
    --build \
      "${srcdir}/${_pkg}_${pkgver}/build/"
}

build()
{
  local \
    _tests_switch=() \
    _tests_switch_status
  _tests_switch=(
    "OFF"
    # "ON"
  )
  for _tests_switch_status \
    in "${_tests_switch[@]}"; do
    _compile \
      "${_tests_switch_status}"
  done
}

check()
{
  _compile \
    "ON"
  "${srcdir}/${_pkg}_${pkgver}/build/test/soltest" \
    -p \
      true -- \
    --testpath \
      "${srcdir}/${_pkg}_${pkgver}/test/"
  _compile \
    "OFF"
}

package() {
  local \
    _binaries=() \
    _bin
  _binaries=(
    "solc"
    "yul-phaser"
  )
  # Assure that the directories exist.
  mkdir \
    -p \
      "${pkgdir}/usr/share/doc/${pkgname}/"
  # Install the software.
  DESTDIR="${pkgdir}/" \
  cmake \
    --install \
      "${srcdir}/${_pkg}_${pkgver}/build/"
  # Rename the binaries because
  # CMAKE_EXECUTABLE_SUFFIX doesn't work
  for _bin in "${_binaries[@]}"; do
    mv \
      "${pkgdir}/usr/bin/${_bin}" \
      "${pkgdir}/usr/bin/${_bin}${pkgver}" || \
      true
  done
  # Install the documentation.
  install \
    -Dm644 \
    "${srcdir}/${_pkg}_${pkgver}/README.md" \
    "${pkgdir}/usr/share/doc/${pkgname}/"
  cp \
    -r \
    "${srcdir}/${_pkg}_${pkgver}/docs/"* \
    "${pkgdir}/usr/share/doc/${pkgname}/"
  find \
    "${pkgdir}/usr/share/doc/${pkgname}/" \
    -type \
      d \
    -exec \
      chmod \
        755 \
	{} +
  find \
    "${pkgdir}/usr/share/doc/${pkgname}/" \
    -type \
      f \
    -exec \
      chmod \
      644 \
      {} +
}
