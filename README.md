#!/bin/bash


set -x  //调试
# Function to display usage information and available options
//展示错误的输出
function display_usage {
    echo "Usage: $0 /path/to/source_directory"
}

# Check if a valid directory path is provided as a command-line argument
//判断是否参数是零个，文件是不是文件夹
if [ $# -eq 0 ] || [ ! -d "$1" ]; then
    echo "Error: Please provide a valid directory path as a command-line argument."
    display_usage
    exit 1
fi

# Directory path of the source directory to be backed up
//获取路径
source_dir="$1"

# Function to create a timestamped backup folder

function create_backup {
    local timestamp=$(date '+%Y-%m-%d_%H-%M-%S')  # Get the current timestamp
    declare -g backup_dir="${source_dir}/backup_${timestamp}"

    # Create the backup folder with the timestamped name
    mkdir "$backup_dir"
    echo "Backup created successfully: $backup_dir"
}

# Function to perform the rotation and keep only the last 3 backups
function perform_rotation {

//sed 's/:$//'正则表达式，表示替换尾部的:为空
    local backups=($(ls -dt "${source_dir}/backup_"* 2>/dev/null |sed 's/:$//' ))  # List existing backups sorted by timestamp

    # Check if there are more than 3 backups
    if [ "${#backups[@]}" -gt 3 ]; then
        local backups_to_remove="${backups[@]:3}"  # Get backups beyond the last 3
        
        name=${backups_to_remove[@]}
        
        rm -rf ${name}  # Remove the oldest backups
    fi
}
function copyto {

 //ls的-I表示过滤不显示backup开头的文件
	for name in $(ls -I "backup*" ${source_dir}/ | sed 's/:$//' 2>/dev/null)
	do
	
		cp ${source_dir}/${name} ${backup_dir}/
	
	done
	
}
# Main script logic
create_backup
perform_rotation
copyto
