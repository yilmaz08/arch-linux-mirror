# Foundation is from: https://gitlab.archlinux.org/archlinux/infrastructure/-/blob/master/roles/syncrepo/files/syncrepo-template.sh
# Always refer to the official Arch Linux guide for the most up-to-date information: https://wiki.archlinux.org/index.php/DeveloperWiki:NewMirrors

### VARIABLES ###
target_dir="" # The directory that everything will be stored at. Example: /srv/http/mirror/archlinux/
sync_repo="" # rsync supporting mirror. Refer to https://wiki.archlinux.org/index.php/DeveloperWiki:NewMirrors for more information on choosing one.
lock_file="/var/lock/syncrepo.lck" # Lock file path
bwlimit=0 # Bandwidth in KiB

[ ! -d "${target_dir}" ] && mkdir -p "${target_dir}" # Create the target directory if it doesn't exist

exec 9>"${lock_file}" # Open lock file
flock -n 9 || exit # Lock file

find "${target_dir}" -name '.~tmp~' -exec rm -r {} + # Remove any leftover .~tmp~ directories

rsync_cmd() {
	local -a cmd=(rsync -rlptH --safe-links --delete-delay --delay-updates
		"--timeout=600" "--contimeout=60" --no-motd)


	if stty &>/dev/null; then
		cmd+=(-h -v --progress) # Interactive
	else
		cmd+=(--quiet) # Non-interactive
	fi

	if ((bwlimit>0)); then # Check if bandwidth limit is set
		cmd+=("--bwlimit=$bwlimit") 
	fi

	"${cmd[@]}" "$@"
}

# You can set the excluded directories as you wish.
# Only mandatory directories are enabled by default in this example.
rsync_cmd \
	--exclude='*.links.tar.gz*' \
	--exclude='/other' \
	--exclude='/sources' \
	--exclude='/images' \
    --exclude='/iso' \
	"${sync_repo}" \
	"${target_dir}"

echo "All Done!"