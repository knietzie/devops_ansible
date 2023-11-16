# devops_ansible
Using Python scripts and a few libraries, we can achieve what we would like to implement.

Write a backup script that:
1. Connects to another server and copies data in an encrypted and compressed format.
2. Allows changing certain parameters when running the script, such as the backup
directory, user, the IP of the remote server, debug mode, directories to copy, and the choice
between full and incremental backup.
3. Utilizes logrotate for rotating outdated versions.
The logic for full and incremental backups includes creating two folders for full and incremental
backups (similarly for incremental):
- Full (Inc): stores the most recent backup.
- FullOld (IncOld): stores the previous backups, which are rotated.


Install `paramiko` library using pip install
pip install `paramiko`


import os

import tarfile

import paramiko

import argparse

import logging

import logging.handlers



# Setup command line arguments

parser = argparse.ArgumentParser(description="Backup Script")

parser.add_argument("--backup-dir", help="Backup directory path", default="backup")

parser.add_argument("--user", help="Username for remote server", required=True)

parser.add_argument("--ip", help="IP address of remote server", required=True)

parser.add_argument("--debug", help="Enable debug mode", action="store_true")

parser.add_argument("--directories", help="Directories to copy", nargs='+', required=True)

parser.add_argument("--backup-type", help="Backup type: full or incremental", choices=['full', 'incremental'], default='full')

args = parser.parse_args()



# Setup logging

log_file = 'backup.log'

logger = logging.getLogger('BackupLogger')

logger.setLevel(logging.DEBUG if args.debug else logging.INFO)

handler = logging.handlers.RotatingFileHandler(log_file, maxBytes=1000000, backupCount=3)

formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')

handler.setFormatter(formatter)

logger.addHandler(handler)



# Establish SSH connection

try:

    transport = paramiko.Transport((args.ip, 22))

    transport.connect(username=args.user, password=input("Enter password: "))

    sftp = transport.open_sftp()



    # Create backup directory if not exists

    if not os.path.exists(args.backup_dir):

        os.makedirs(args.backup_dir)



    # Backup logic

    backup_filename = f"{args.backup_type}_backup.tar.gz"

    with tarfile.open(os.path.join(args.backup_dir, backup_filename), 'w:gz') as tar:

        for directory in args.directories:

            tar.add(directory)



    # Transfer backup file

    sftp.put(os.path.join(args.backup_dir, backup_filename), backup_filename)



    # Clean up local backup file

    os.remove(os.path.join(args.backup_dir, backup_filename))

    logger.info("Backup completed successfully!")



except Exception as e:

    logger.error(f"Backup failed: {str(e)}")

finally:

    # Close connections

    sftp.close()

    transport.close()



