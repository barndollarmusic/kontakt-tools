# CC Split to Keyswitch

Turn input MIDI CC messages from your controller hardware and DAW into keyswitch notes, optionally
with split behavior depending on the value. This lets you control velocity-sensitive keyswitches,
for example.

<!-- TODO: Add screenshot or animated GIF. -->
<!-- TODO: Add link to YouTube video showing it. -->

## Setup Instructions

1. In the code block below this list, click on the copy button that appears in the upper right
   corner.

2. Open Kontakt (or Kontakt Player) and click the **KSP** button toward the top.

3. TODO: Finish these instructions.

```text
{***********************************************************
CC Split to Keyswitch
https://github.com/barndollarmusic/kontakt-tools

Author: Eric Barndollar
Modified: 2023-07-07
License: MIT
************************************************************}

on init
	message("")
	set_script_title("CC Split to Keyswitch")
	set_ui_height(3)

	{* Temporary variables. *}
	declare $part
	declare $affected_chan_idx
	declare $out_note
	declare $out_velocity

	{* Top row. *}
	declare ui_label $channels_label (1, 1)
	set_text($channels_label, "Chan:")
	set_control_par(get_ui_id($channels_label), $CONTROL_PAR_WIDTH, 25)
	set_control_par(get_ui_id($channels_label), $CONTROL_PAR_POS_X, 50)
	set_control_par(get_ui_id($channels_label), $CONTROL_PAR_POS_Y, 2)
	set_control_par(get_ui_id($channels_label), $CONTROL_PAR_HIDE, $HIDE_PART_BG)

	declare ui_menu $channels_menu
	add_menu_item($channels_menu, "All", 0)
	add_menu_item($channels_menu, "1", 1)
	add_menu_item($channels_menu, "2", 2)
	add_menu_item($channels_menu, "3", 3)
	add_menu_item($channels_menu, "4", 4)
	add_menu_item($channels_menu, "5", 5)
	add_menu_item($channels_menu, "6", 6)
	add_menu_item($channels_menu, "7", 7)
	add_menu_item($channels_menu, "8", 8)
	add_menu_item($channels_menu, "9", 9)
	add_menu_item($channels_menu, "10", 10)
	add_menu_item($channels_menu, "11", 11)
	add_menu_item($channels_menu, "12", 12)
	add_menu_item($channels_menu, "13", 13)
	add_menu_item($channels_menu, "14", 14)
	add_menu_item($channels_menu, "15", 15)
	add_menu_item($channels_menu, "16", 16)
	set_control_par(get_ui_id($channels_menu), $CONTROL_PAR_WIDTH, 60)
	set_control_par(get_ui_id($channels_menu), $CONTROL_PAR_POS_X, 84)
	set_control_par(get_ui_id($channels_menu), $CONTROL_PAR_POS_Y, 2)
	make_persistent($channels_menu)
	read_persistent_var($channels_menu)

	declare ui_label $input_label (3, 1)
	set_text($input_label, "Input CC Values (inclusive)")
	set_control_par(get_ui_id($input_label), $CONTROL_PAR_GRID_X, 2)
	set_control_par(get_ui_id($input_label), $CONTROL_PAR_GRID_Y, 1)

	declare ui_label $output_label (2, 1)
	set_text($output_label, "Output Keyswitch")
	set_control_par(get_ui_id($output_label), $CONTROL_PAR_GRID_X, 5)
	set_control_par(get_ui_id($output_label), $CONTROL_PAR_GRID_Y, 1)

	{* First CC value range. *}
	declare ui_switch $first_enabled
	set_control_par(get_ui_id($first_enabled), $CONTROL_PAR_VALUE, 1)
	set_control_par(get_ui_id($first_enabled), $CONTROL_PAR_POS_X, 130)
	set_control_par(get_ui_id($first_enabled), $CONTROL_PAR_POS_Y, 25)
	set_control_par(get_ui_id($first_enabled), $CONTROL_PAR_WIDTH, 14)
	set_control_par(get_ui_id($first_enabled), $CONTROL_PAR_HEIGHT, 14)
	set_control_par_str(get_ui_id($first_enabled), $CONTROL_PAR_TEXT, "")
	make_persistent($first_enabled)
	read_persistent_var($first_enabled)

	declare ui_value_edit $first_cc (0, 127, 1)
	set_control_par(get_ui_id($first_cc), $CONTROL_PAR_VALUE, 20)
	set_control_par(get_ui_id($first_cc), $CONTROL_PAR_GRID_X, 2)
	set_control_par(get_ui_id($first_cc), $CONTROL_PAR_GRID_Y, 2)
	set_control_par_str(get_ui_id($first_cc), $CONTROL_PAR_TEXT, "CC#")
	make_persistent($first_cc)
	read_persistent_var($first_cc)

	declare ui_value_edit $first_cc_val_min (0, 127, 1)
	set_control_par(get_ui_id($first_cc_val_min), $CONTROL_PAR_VALUE, 0)
	set_control_par(get_ui_id($first_cc_val_min), $CONTROL_PAR_GRID_X, 3)
	set_control_par(get_ui_id($first_cc_val_min), $CONTROL_PAR_GRID_Y, 2)
	set_control_par_str(get_ui_id($first_cc_val_min), $CONTROL_PAR_TEXT, "Min")
	make_persistent($first_cc_val_min)
	read_persistent_var($first_cc_val_min)

	declare ui_value_edit $first_cc_val_max (0, 127, 1)
	set_control_par(get_ui_id($first_cc_val_max), $CONTROL_PAR_VALUE, 127)
	set_control_par(get_ui_id($first_cc_val_max), $CONTROL_PAR_GRID_X, 4)
	set_control_par(get_ui_id($first_cc_val_max), $CONTROL_PAR_GRID_Y, 2)
	set_control_par_str(get_ui_id($first_cc_val_max), $CONTROL_PAR_TEXT, "Max")
	make_persistent($first_cc_val_max)
	read_persistent_var($first_cc_val_max)

	declare ui_value_edit $first_note (0, 127, $VALUE_EDIT_MODE_NOTE_NAMES)
	set_control_par(get_ui_id($first_note), $CONTROL_PAR_VALUE, 0)
	set_control_par(get_ui_id($first_note), $CONTROL_PAR_GRID_X, 5)
	set_control_par(get_ui_id($first_note), $CONTROL_PAR_GRID_Y, 2)
	set_control_par_str(get_ui_id($first_note), $CONTROL_PAR_TEXT, "Note")
	make_persistent($first_note)
	read_persistent_var($first_note)

	declare ui_value_edit $first_velocity (0, 127, 1)
	set_control_par(get_ui_id($first_velocity), $CONTROL_PAR_VALUE, 127)
	set_control_par(get_ui_id($first_velocity), $CONTROL_PAR_GRID_X, 6)
	set_control_par(get_ui_id($first_velocity), $CONTROL_PAR_GRID_Y, 2)
	set_control_par_str(get_ui_id($first_velocity), $CONTROL_PAR_TEXT, "Velocity")
	make_persistent($first_velocity)
	read_persistent_var($first_velocity)

	{* Second CC value range. *}
	declare ui_switch $second_enabled
	set_control_par(get_ui_id($second_enabled), $CONTROL_PAR_VALUE, 0)
	set_control_par(get_ui_id($second_enabled), $CONTROL_PAR_POS_X, 130)
	set_control_par(get_ui_id($second_enabled), $CONTROL_PAR_POS_Y, 46)
	set_control_par(get_ui_id($second_enabled), $CONTROL_PAR_WIDTH, 14)
	set_control_par(get_ui_id($second_enabled), $CONTROL_PAR_HEIGHT, 14)
	set_control_par_str(get_ui_id($second_enabled), $CONTROL_PAR_TEXT, "")
	make_persistent($second_enabled)
	read_persistent_var($second_enabled)

	declare ui_value_edit $second_cc (0, 127, 1)
	set_control_par(get_ui_id($second_cc), $CONTROL_PAR_VALUE, 21)
	set_control_par(get_ui_id($second_cc), $CONTROL_PAR_GRID_X, 2)
	set_control_par(get_ui_id($second_cc), $CONTROL_PAR_GRID_Y, 3)
	set_control_par_str(get_ui_id($second_cc), $CONTROL_PAR_TEXT, "CC#")
	make_persistent($second_cc)
	read_persistent_var($second_cc)

	declare ui_value_edit $second_cc_val_min (0, 127, 1)
	set_control_par(get_ui_id($second_cc_val_min), $CONTROL_PAR_VALUE, 0)
	set_control_par(get_ui_id($second_cc_val_min), $CONTROL_PAR_GRID_X, 3)
	set_control_par(get_ui_id($second_cc_val_min), $CONTROL_PAR_GRID_Y, 3)
	set_control_par_str(get_ui_id($second_cc_val_min), $CONTROL_PAR_TEXT, "Min")
	make_persistent($second_cc_val_min)
	read_persistent_var($second_cc_val_min)

	declare ui_value_edit $second_cc_val_max (0, 127, 1)
	set_control_par(get_ui_id($second_cc_val_max), $CONTROL_PAR_VALUE, 127)
	set_control_par(get_ui_id($second_cc_val_max), $CONTROL_PAR_GRID_X, 4)
	set_control_par(get_ui_id($second_cc_val_max), $CONTROL_PAR_GRID_Y, 3)
	set_control_par_str(get_ui_id($second_cc_val_max), $CONTROL_PAR_TEXT, "Max")
	make_persistent($second_cc_val_max)
	read_persistent_var($second_cc_val_max)

	declare ui_value_edit $second_note (0, 127, $VALUE_EDIT_MODE_NOTE_NAMES)
	set_control_par(get_ui_id($second_note), $CONTROL_PAR_VALUE, 1)
	set_control_par(get_ui_id($second_note), $CONTROL_PAR_GRID_X, 5)
	set_control_par(get_ui_id($second_note), $CONTROL_PAR_GRID_Y, 3)
	set_control_par_str(get_ui_id($second_note), $CONTROL_PAR_TEXT, "Note")
	make_persistent($second_note)
	read_persistent_var($second_note)

	declare ui_value_edit $second_velocity (0, 127, 1)
	set_control_par(get_ui_id($second_velocity), $CONTROL_PAR_VALUE, 127)
	set_control_par(get_ui_id($second_velocity), $CONTROL_PAR_GRID_X, 6)
	set_control_par(get_ui_id($second_velocity), $CONTROL_PAR_GRID_Y, 3)
	set_control_par_str(get_ui_id($second_velocity), $CONTROL_PAR_TEXT, "Velocity")
	make_persistent($second_velocity)
	read_persistent_var($second_velocity)

	{* Third CC value range. *}
	declare ui_switch $third_enabled
	set_control_par(get_ui_id($third_enabled), $CONTROL_PAR_VALUE, 0)
	set_control_par(get_ui_id($third_enabled), $CONTROL_PAR_POS_X, 130)
	set_control_par(get_ui_id($third_enabled), $CONTROL_PAR_POS_Y, 67)
	set_control_par(get_ui_id($third_enabled), $CONTROL_PAR_WIDTH, 14)
	set_control_par(get_ui_id($third_enabled), $CONTROL_PAR_HEIGHT, 14)
	set_control_par_str(get_ui_id($third_enabled), $CONTROL_PAR_TEXT, "")
	make_persistent($third_enabled)
	read_persistent_var($third_enabled)

	declare ui_value_edit $third_cc (0, 127, 1)
	set_control_par(get_ui_id($third_cc), $CONTROL_PAR_VALUE, 22)
	set_control_par(get_ui_id($third_cc), $CONTROL_PAR_GRID_X, 2)
	set_control_par(get_ui_id($third_cc), $CONTROL_PAR_GRID_Y, 4)
	set_control_par_str(get_ui_id($third_cc), $CONTROL_PAR_TEXT, "CC#")
	make_persistent($third_cc)
	read_persistent_var($third_cc)

	declare ui_value_edit $third_cc_val_min (0, 127, 1)
	set_control_par(get_ui_id($third_cc_val_min), $CONTROL_PAR_VALUE, 0)
	set_control_par(get_ui_id($third_cc_val_min), $CONTROL_PAR_GRID_X, 3)
	set_control_par(get_ui_id($third_cc_val_min), $CONTROL_PAR_GRID_Y, 4)
	set_control_par_str(get_ui_id($third_cc_val_min), $CONTROL_PAR_TEXT, "Min")
	make_persistent($third_cc_val_min)
	read_persistent_var($third_cc_val_min)

	declare ui_value_edit $third_cc_val_max (0, 127, 1)
	set_control_par(get_ui_id($third_cc_val_max), $CONTROL_PAR_VALUE, 127)
	set_control_par(get_ui_id($third_cc_val_max), $CONTROL_PAR_GRID_X, 4)
	set_control_par(get_ui_id($third_cc_val_max), $CONTROL_PAR_GRID_Y, 4)
	set_control_par_str(get_ui_id($third_cc_val_max), $CONTROL_PAR_TEXT, "Max")
	make_persistent($third_cc_val_max)
	read_persistent_var($third_cc_val_max)

	declare ui_value_edit $third_note (0, 127, $VALUE_EDIT_MODE_NOTE_NAMES)
	set_control_par(get_ui_id($third_note), $CONTROL_PAR_VALUE, 2)
	set_control_par(get_ui_id($third_note), $CONTROL_PAR_GRID_X, 5)
	set_control_par(get_ui_id($third_note), $CONTROL_PAR_GRID_Y, 4)
	set_control_par_str(get_ui_id($third_note), $CONTROL_PAR_TEXT, "Note")
	make_persistent($third_note)
	read_persistent_var($third_note)

	declare ui_value_edit $third_velocity (0, 127, 1)
	set_control_par(get_ui_id($third_velocity), $CONTROL_PAR_VALUE, 127)
	set_control_par(get_ui_id($third_velocity), $CONTROL_PAR_GRID_X, 6)
	set_control_par(get_ui_id($third_velocity), $CONTROL_PAR_GRID_Y, 4)
	set_control_par_str(get_ui_id($third_velocity), $CONTROL_PAR_TEXT, "Velocity")
	make_persistent($third_velocity)
	read_persistent_var($third_velocity)

	{* Fourth CC value range. *}
	declare ui_switch $fourth_enabled
	set_control_par(get_ui_id($fourth_enabled), $CONTROL_PAR_VALUE, 0)
	set_control_par(get_ui_id($fourth_enabled), $CONTROL_PAR_POS_X, 130)
	set_control_par(get_ui_id($fourth_enabled), $CONTROL_PAR_POS_Y, 88)
	set_control_par(get_ui_id($fourth_enabled), $CONTROL_PAR_WIDTH, 14)
	set_control_par(get_ui_id($fourth_enabled), $CONTROL_PAR_HEIGHT, 14)
	set_control_par_str(get_ui_id($fourth_enabled), $CONTROL_PAR_TEXT, "")
	make_persistent($fourth_enabled)
	read_persistent_var($fourth_enabled)

	declare ui_value_edit $fourth_cc (0, 127, 1)
	set_control_par(get_ui_id($fourth_cc), $CONTROL_PAR_VALUE, 23)
	set_control_par(get_ui_id($fourth_cc), $CONTROL_PAR_GRID_X, 2)
	set_control_par(get_ui_id($fourth_cc), $CONTROL_PAR_GRID_Y, 5)
	set_control_par_str(get_ui_id($fourth_cc), $CONTROL_PAR_TEXT, "CC#")
	make_persistent($fourth_cc)
	read_persistent_var($fourth_cc)

	declare ui_value_edit $fourth_cc_val_min (0, 127, 1)
	set_control_par(get_ui_id($fourth_cc_val_min), $CONTROL_PAR_VALUE, 0)
	set_control_par(get_ui_id($fourth_cc_val_min), $CONTROL_PAR_GRID_X, 3)
	set_control_par(get_ui_id($fourth_cc_val_min), $CONTROL_PAR_GRID_Y, 5)
	set_control_par_str(get_ui_id($fourth_cc_val_min), $CONTROL_PAR_TEXT, "Min")
	make_persistent($fourth_cc_val_min)
	read_persistent_var($fourth_cc_val_min)

	declare ui_value_edit $fourth_cc_val_max (0, 127, 1)
	set_control_par(get_ui_id($fourth_cc_val_max), $CONTROL_PAR_VALUE, 127)
	set_control_par(get_ui_id($fourth_cc_val_max), $CONTROL_PAR_GRID_X, 4)
	set_control_par(get_ui_id($fourth_cc_val_max), $CONTROL_PAR_GRID_Y, 5)
	set_control_par_str(get_ui_id($fourth_cc_val_max), $CONTROL_PAR_TEXT, "Max")
	make_persistent($fourth_cc_val_max)
	read_persistent_var($fourth_cc_val_max)

	declare ui_value_edit $fourth_note (0, 127, $VALUE_EDIT_MODE_NOTE_NAMES)
	set_control_par(get_ui_id($fourth_note), $CONTROL_PAR_VALUE, 3)
	set_control_par(get_ui_id($fourth_note), $CONTROL_PAR_GRID_X, 5)
	set_control_par(get_ui_id($fourth_note), $CONTROL_PAR_GRID_Y, 5)
	set_control_par_str(get_ui_id($fourth_note), $CONTROL_PAR_TEXT, "Note")
	make_persistent($fourth_note)
	read_persistent_var($fourth_note)

	declare ui_value_edit $fourth_velocity (0, 127, 1)
	set_control_par(get_ui_id($fourth_velocity), $CONTROL_PAR_VALUE, 127)
	set_control_par(get_ui_id($fourth_velocity), $CONTROL_PAR_GRID_X, 6)
	set_control_par(get_ui_id($fourth_velocity), $CONTROL_PAR_GRID_Y, 5)
	set_control_par_str(get_ui_id($fourth_velocity), $CONTROL_PAR_TEXT, "Velocity")
	make_persistent($fourth_velocity)
	read_persistent_var($fourth_velocity)

	{* Show or hide controls based on current enabled values. *}
	if (get_control_par(get_ui_id($first_enabled), $CONTROL_PAR_VALUE) = 0)
		$part := $HIDE_WHOLE_CONTROL
	else
		$part := $HIDE_PART_NOTHING
	end if

	set_control_par(get_ui_id($first_cc), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($first_cc_val_min), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($first_cc_val_max), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($first_note), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($first_velocity), $CONTROL_PAR_HIDE, $part)

	if (get_control_par(get_ui_id($second_enabled), $CONTROL_PAR_VALUE) = 0)
		$part := $HIDE_WHOLE_CONTROL
	else
		$part := $HIDE_PART_NOTHING
	end if

	set_control_par(get_ui_id($second_cc), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($second_cc_val_min), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($second_cc_val_max), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($second_note), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($second_velocity), $CONTROL_PAR_HIDE, $part)

	if (get_control_par(get_ui_id($third_enabled), $CONTROL_PAR_VALUE) = 0)
		$part := $HIDE_WHOLE_CONTROL
	else
		$part := $HIDE_PART_NOTHING
	end if

	set_control_par(get_ui_id($third_cc), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($third_cc_val_min), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($third_cc_val_max), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($third_note), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($third_velocity), $CONTROL_PAR_HIDE, $part)

	if (get_control_par(get_ui_id($fourth_enabled), $CONTROL_PAR_VALUE) = 0)
		$part := $HIDE_WHOLE_CONTROL
	else
		$part := $HIDE_PART_NOTHING
	end if

	set_control_par(get_ui_id($fourth_cc), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($fourth_cc_val_min), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($fourth_cc_val_max), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($fourth_note), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($fourth_velocity), $CONTROL_PAR_HIDE, $part)
end on

on ui_control ($first_enabled)
	if (get_control_par(get_ui_id($first_enabled), $CONTROL_PAR_VALUE) = 0)
		$part := $HIDE_WHOLE_CONTROL
	else
		$part := $HIDE_PART_NOTHING
	end if

	set_control_par(get_ui_id($first_cc), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($first_cc_val_min), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($first_cc_val_max), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($first_note), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($first_velocity), $CONTROL_PAR_HIDE, $part)
end on

on ui_control ($second_enabled)
	if (get_control_par(get_ui_id($second_enabled), $CONTROL_PAR_VALUE) = 0)
		$part := $HIDE_WHOLE_CONTROL
	else
		$part := $HIDE_PART_NOTHING
	end if

	set_control_par(get_ui_id($second_cc), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($second_cc_val_min), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($second_cc_val_max), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($second_note), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($second_velocity), $CONTROL_PAR_HIDE, $part)
end on

on ui_control ($third_enabled)
	if (get_control_par(get_ui_id($third_enabled), $CONTROL_PAR_VALUE) = 0)
		$part := $HIDE_WHOLE_CONTROL
	else
		$part := $HIDE_PART_NOTHING
	end if

	set_control_par(get_ui_id($third_cc), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($third_cc_val_min), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($third_cc_val_max), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($third_note), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($third_velocity), $CONTROL_PAR_HIDE, $part)
end on

on ui_control ($fourth_enabled)
	if (get_control_par(get_ui_id($fourth_enabled), $CONTROL_PAR_VALUE) = 0)
		$part := $HIDE_WHOLE_CONTROL
	else
		$part := $HIDE_PART_NOTHING
	end if

	set_control_par(get_ui_id($fourth_cc), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($fourth_cc_val_min), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($fourth_cc_val_max), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($fourth_note), $CONTROL_PAR_HIDE, $part)
	set_control_par(get_ui_id($fourth_velocity), $CONTROL_PAR_HIDE, $part)
end on

on midi_in
	{Only process CC messages...}
	if ($MIDI_COMMAND # $MIDI_COMMAND_CC)
		exit
	end if

	{...on the configured channel(s)...}
	$affected_chan_idx := get_control_par(get_ui_id($channels_menu), $CONTROL_PAR_SELECTED_ITEM_IDX) - 1
	if (($affected_chan_idx # -1) and ($MIDI_CHANNEL # $affected_chan_idx))
		exit
	end if

	{...if they match one of the configured inputs...}
	$out_note := -1
	$out_velocity := 0

	if (($first_enabled = 1) and ($MIDI_BYTE_1 = $first_cc) and in_range($MIDI_BYTE_2, $first_cc_val_min, $first_cc_val_max))
		$out_note := $first_note
		$out_velocity := $first_velocity
	end if

	if (($out_note = -1) and ($second_enabled = 1) and ($MIDI_BYTE_1 = $second_cc) and in_range($MIDI_BYTE_2, $second_cc_val_min, $second_cc_val_max))
		$out_note := $second_note
		$out_velocity := $second_velocity
	end if
	
	if (($out_note = -1) and ($third_enabled = 1) and ($MIDI_BYTE_1 = $third_cc) and in_range($MIDI_BYTE_2, $third_cc_val_min, $third_cc_val_max))
		$out_note := $third_note
		$out_velocity := $third_velocity
	end if
	
	if (($out_note = -1) and ($fourth_enabled = 1) and ($MIDI_BYTE_1 = $fourth_cc) and in_range($MIDI_BYTE_2, $fourth_cc_val_min, $fourth_cc_val_max))
		$out_note := $fourth_note
		$out_velocity := $fourth_velocity
	end if

	if ($out_note = -1)
		exit
	end if

	{* Got a match! Output the keyswitch. *}
	ignore_midi  {* Swallow the CC. *}

	set_midi($MIDI_CHANNEL, $MIDI_COMMAND_NOTE_ON, $out_note, $out_velocity)
	set_midi($MIDI_CHANNEL, $MIDI_COMMAND_NOTE_OFF, $out_note, 0)
end on
```

## Usage Instructions

TODO.
