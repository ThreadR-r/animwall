#!/bin/sh
#
# The GPLv2 License
#
#   Copyright (C) 2019  Peter Kenji Yamanaka
#
#   This program is free software; you can redistribute it and/or modify
#   it under the terms of the GNU General Public License as published by
#   the Free Software Foundation; either version 2 of the License, or
#   (at your option) any later version.
#
#   This program is distributed in the hope that it will be useful,
#   but WITHOUT ANY WARRANTY; without even the implied warranty of
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#   GNU General Public License for more details.
#
#   You should have received a copy of the GNU General Public License along
#   with this program; if not, write to the Free Software Foundation, Inc.,
#   51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

# Lowest priority, super nice (make this number smaller to increase priority, cannot go below 0)
__nice_value=19

# PID list, do not touch
__pids=""

# $1 wallpaper
# $2 monitor coords
wrapped()
{
  exec nice -n "${__nice_value}" xwinwrap \
    -ni \
    -un \
    -st \
    -fdt \
    -sp \
    -nf \
    -b \
    -g "$2" \
    -- nice -n "${__nice_value}" mpv \
    -wid WID "$1" \
    --geometry="$2" \
    --no-osc \
    --no-osd-bar \
    --loop-file \
    --player-operation-mode=cplayer \
    --no-audio \
    --panscan=1.0 \
    --no-keepaspect \
    --no-input-default-bindings \
    --hwdec
}

cancel()
{
  # Kill runaway pids
  for p in ${__pids}; do
    p="$(printf -- "%s" "${p}" \
      | sed 's/^[[:blank:]]*//' | sed 's/[[:blank:]]*$//')"
    if [ -n "${p}" ]; then
      kill "${p}" > /dev/null 2>&1
    fi
  done

  # Reset terminal in case it bugs out
  reset
}

main()
{
  if ! command -v mpv > /dev/null; then
    printf -- 'You must have "mpv" installed.\n'
    return 1
  fi

  if ! command -v xwinwrap > /dev/null; then
    printf -- 'You must have "xwinwrap" installed.\n'
    return 1
  fi

  if ! command -v nice > /dev/null; then
    printf -- 'You must have "nice" installed.\n'
    return 1
  fi

  # Trap return signals
  trap cancel INT TERM

  # Check for config file
  animwall_configs=""
  config_location="${XDG_CONFIG_DIR:-"${HOME}/.config"}/animwall.conf"
  if [ -f "${config_location}" ] && [ -r "${config_location}" ]; then
    printf -- 'Reading animwall.conf from %s\n' "${config_location}"
    animwall_configs="$(cat -- "${config_location}")"
  else
    printf -- 'Could not find animwall.conf file at %s\n' "${config_location}"
    printf -- 'Creating example file...\n'
    printf -- '%s\n' "$(cat << EOF
~/wallpapers/example_1.mp4:1920x1080+0+0
~/pictures/example2.jpg:2560x1440+1920+0
EOF
)" > "${config_location}"
    return 1
  fi

  if [ -z "${animwall_configs}" ]; then
    printf -- 'Animwall config was empty!\n'
    return 1
  fi

  # Parse config file
  for config in ${animwall_configs}; do
    split_config="$(printf -- "%s" "${config}" | tr ':' ' ')"
    wallpaper="$(printf -- "%s" "${split_config}" | awk '{ print $1 }')"
    monitor="$(printf -- "%s" "${split_config}" | awk '{ print $2 }')"

    # In case the wallpaper path has ~ as HOME, we replace it
    wallpaper="$(printf -- "%s" "${wallpaper}" | sed "s|^\~|$HOME|")"

    if [ -n "${wallpaper}" ] && [ -n "${monitor}" ]; then
      wrapped "${wallpaper}" "${monitor}" &
      __pids="$! ${__pids}"
    else
      if [ -z "${wallpaper}" ]; then
        printf -- 'Missing wallpaper path in config: %s\n' "${config}"
      fi

      if [ -z "${monitor}" ]; then
        printf -- 'Missing monitor geometry in config: %s\n' "${config}"
      fi
    fi

    unset split_config
    unset wallpaper
    unset monitor
    unset config
  done
  unset config

  # Wait for all wrapped processes to finish
  #
  # Do not quote so that this will split into a list, instead of a single,
  # space separated input
  # shellcheck disable=SC2086
  wait ${__pids}
}

main "$@"
