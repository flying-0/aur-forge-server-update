# Maintainer: thisischrys <thisischrys+aur at gmail dot com>
# Maintainer: Nitroretro <nitroretro@protonmail.com>
# Contributor: flying <flyinghat42@gmail.com>

# Based on the `minecraft-server` AUR package by:
## Maintainer: Gordian Edenhofer <gordian.edenhofer@gmail.com>
## Contributor: Philip Abernethy <chais.z3r0@gmail.com>
## Contributor: sowieso <sowieso@dukun.de>

_ver="1.18.1_39.0.10-1"
_minecraft_ver_latest="1.18.1"

IFS="-" read -ra _ver_temp <<< "$_ver"
IFS="_" read -ra _pkgver_temp <<< "${_ver_temp[0]}"
IFS="." read -ra _minecraft_ver_temp <<< "${_pkgver_temp[0]}"

_minecraft_ver=${_pkgver_temp[0]}
_minecraft_ver_major=${_minecraft_ver_temp[0]:-0}
_minecraft_ver_minor=${_minecraft_ver_temp[1]:-0}
_minecraft_ver_patch=${_minecraft_ver_temp[2]:-0}
_forge_ver=${_pkgver_temp[1]}

_pkgver=${_ver_temp[0]//_/-}

if [ "$_minecraft_ver" = "$_minecraft_ver_latest" ]; then
	pkgname="forge-server"
	_forge_name="forge"
else
	pkgname="forge-server-${_minecraft_ver}"
	_forge_name="forge-${_minecraft_ver}"
fi
pkgver=${_ver_temp[0]}
pkgrel=${_ver_temp[1]}
_mng_ver=1.0.0
pkgdesc="Minecraft Forge server unit files, script and jar"
arch=("any")
url="https://minecraftforge.net"
license=("custom")
depends=("java-runtime-headless>=17" "tmux" "sudo" "bash" "awk" "sed")
optdepends=("tar: required in order to create world backups"
	"netcat: required in order to suspend an idle server")
provides=("forge-server=${pkgver}")
backup=("etc/conf.d/${_forge_name}")
install="${pkgname}.install"

source=("minecraft-server-${_mng_ver}.tar.gz"::"https://github.com/Edenhofer/minecraft-server/archive/refs/tags/v${_mng_ver}.tar.gz"
		"forge-${_pkgver}-installer.jar"::"https://maven.minecraftforge.net/net/minecraftforge/forge/${_pkgver}/forge-${_pkgver}-installer.jar")

noextract=("forge-${_pkgver}.jar")
sha512sums=('e315277da81cb28de338e870f477dc58dc9d8f8542594431ab5321150c92ff5634ace2be8c6778d1edb718fdeb6850d7021bffcbd3cae2a00f20e3a64caa3d92'
			'df0705e2fb20cb03638771d6516857273bd16d191f14a0adfe517996938957bb565184725f0bf06295da0cd132e469933fe5fd1012657fdd6b2fb3e706d14dcf'
			'3da10d63a5edee4bc8bcd3d5c2730771062f7fa58626a8c51635fbe96bfbceca3ff6937cfaad3e17f16a94ef95137f7c78cc6dac1c846a6b9a8f18d3c6355973')

# -- License -- #
_licenses=()
_license_suffix="-${pkgname}-${_pkgver}.txt"
_branch="${_minecraft_ver_major}.${_minecraft_ver_minor}.x"

_add_license() {
	_path=$1
	_repo=${2:-MinecraftForge}
	_github_branch=${3:-"$_branch"}
	_filename="$(basename "$_path")${_license_suffix}"

	_licenses+=("$_filename")
	source+=("$_filename"::"https://raw.githubusercontent.com/MinecraftForge/${_repo}/${_github_branch}/${_path}.txt")
}

_add_license "LICENSE"
# -- /License -- #

_game="forge"
_server_root="/srv/${_forge_name}"
_server_jar_dir="${_server_root}/libraries/net/minecraftforge/forge/${_pkgver}"

prepare() {
	java -jar "forge-${_pkgver}-installer.jar" --installServer
}

build() {
	make -C "${srcdir}/minecraft-server-${_mng_ver}" clean

	make -C "${srcdir}/minecraft-server-${_mng_ver}" \
		GAME=${_game} \
		MYNAME=${_game}d \
		SERVER_ROOT=${_server_root} \
		BACKUP_PATHS="world" \
		GAME_USER=${_game} \
		MAIN_EXECUTABLE=forge-${_pkgver}-server.jar \
		SERVER_START_CMD="java @${_server_jar_dir}/user_jvm_args.txt @${_server_jar_dir}/unix_args.txt" \
		all
}

package() {
	make -C "${srcdir}/minecraft-server-${_mng_ver}" \
		DESTDIR="${pkgdir}" \
		GAME=${_game} \
		MYNAME=${_game}d \
		install

	# Install Forge
	find libraries -type f -print0 | xargs -0 -i@ install -Dm644 "@" "${pkgdir}${_server_root}/@"

	# Install arguments file and do some housekeeping
	install -m644 "${srcdir}/user_jvm_args.txt" "${pkgdir}${_server_jar_dir}/user_jvm_args.txt"
	echo "$(awk 'NR==15{print "# Custom JVM arguments may be added to user_jvm_args.txt or directly to this file."}1' ${pkgdir}/etc/conf.d/${_game})" > ${pkgdir}/etc/conf.d/${_game}
	echo "$(awk 'NR==16{print "# Typical Default Arguments: -Xms512M -Xmx1024M @{unix_args.txt location}"}1' ${pkgdir}/etc/conf.d/${_game})" > ${pkgdir}/etc/conf.d/${_game}
	rm "${pkgdir}${_server_jar_dir}/win_args.txt"

	# Link log files
	mkdir -p "${pkgdir}/var/log/"
	install -dm2755 "${pkgdir}/${_server_root}/logs"
	ln -s "${_server_root}/logs" "${pkgdir}/var/log/${_forge_name}"

	# Install license
	for _license in "${_licenses[@]}"; do
		_filename="$(basename "$_license" "$_license_suffix").txt"
		_filename="${_filename//\%20/ }"
		install -Dm644 "$_license" "${pkgdir}/usr/share/licenses/${pkgname}/$_filename"
	done

	chmod g+ws "${pkgdir}${_server_root}"
}
