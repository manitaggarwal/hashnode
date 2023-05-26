---
title: "How to Automate Backups with Go"
datePublished: Fri May 26 2023 06:39:23 GMT+0000 (Coordinated Universal Time)
cuid: cli470ai0001209mchko76yit
slug: how-to-automate-backups-with-go
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/fd9mIBluHkA/upload/8cdd7a2388c5d2069e08b4e496af58cf.jpeg
tags: go, learning, backup

---

In today's digital age, data loss can be catastrophic for individuals and businesses alike. Having a reliable backup system in place is crucial to ensure the safety and availability of your important files and documents. Let's discuss how to automate backups using the Go programming language.

### **Prerequisites**

Before we dive into the backup script, let's make sure we have the necessary prerequisites set up:

1. Go Installation: Make sure you have Go installed on your machine. You can download it from the official Go website ([**https://golang.org**](https://golang.org)) and follow the installation instructions for your operating system.
    
2. External Dependencies: Ensure that the `tar` and `scp` (if using `scp` backup) or `rclone` (if using `rclone` backup) commands are available on your system. These commands are used by the backup script to create the backup archive and copy it to the destination server.
    

### **Getting Started**

Now that we have the prerequisites in place, let's start writing the backup script in Go. Open your favorite text editor or IDE and create a new file called `main.go`. Copy the following code into the file:

```go
package main

import (
	"fmt"
	"log"
	"os"
	"os/exec"
	"time"

	"github.com/spf13/viper"
)

func main() {

	// read values from configuration file
	configPath := os.Getenv("BACKUP_CONFIG_PATH")
	if configPath == "" {
		log.Fatalf("missing BACKUP_CONFIG_PATH environment variable")
		os.Exit(1)
	}
	viper.SetConfigFile(configPath)
	if err := viper.ReadInConfig(); err != nil {
		fmt.Printf("Failed to read configuration file: %v\n", err)
		os.Exit(1)
	}

	// replace these values with your own directory path and server details
	dirPath := viper.GetString("dirPath")
	server := viper.GetString("server")
	backupType := viper.GetString("backupType")

	archiveFileName := viper.GetString("archiveFileName") + "_" +
		time.Now().Format("2006-01-02_15-04") + ".tar.gz"

	// create a tar archive of the directory
	tarCmd := exec.Command("tar", "-czf", archiveFileName, dirPath)
	if err := tarCmd.Run(); err != nil {
		fmt.Printf("Failed to create tar archive: %v\n", err)
		os.Exit(1)
	}

	if backupType == "scp" {
		// copy the archive to the server via scp
		if err := copyArchiveToServer(server, archiveFileName); err != nil {
			fmt.Printf("Failed to copy archive to server: %v\n", err)
			removeArchive(archiveFileName)
			os.Exit(1)
		}
	} else if backupType == "rclone" {
		// copy the archive to the new server via rclone
		rcloneCmd := exec.Command("rclone", "copy", archiveFileName, server)
		if err := rcloneCmd.Run(); err != nil {
			fmt.Printf("Failed to copy archive to object storage: %v\n", err)
			removeArchive(archiveFileName)
			os.Exit(1)
		}
	} else {
		fmt.Printf("Invalid backup type: %v\n, can be scp or rclone", backupType)
		removeArchive(archiveFileName)
		os.Exit(1)
	}

	// delete the local archive file
	if err := removeArchive(archiveFileName); err != nil {
		fmt.Printf("Failed to delete local archive file: %v\n", err)
		os.Exit(1)
	}

	fmt.Println("Archive created and copied to new server successfully.")
}

func copyArchiveToServer(server string, file string) error {
	scpCmd := exec.Command("scp", file, server)
	if err := scpCmd.Run(); err != nil {
		return fmt.Errorf("failed to copy archive to new server: %v", err)
	}
	return nil
}

func removeArchive(file string) error {
	if err := os.Remove(file); err != nil {
		return fmt.Errorf("failed to delete %v: %v", file, err)
	}
	return nil
}
```

### **Understanding the Script**

Let's take a moment to understand the different parts of the backup script.

1. Configuration File: The script expects the path to the configuration file to be set in the `BACKUP_CONFIG_PATH` environment variable. It uses the Viper library to read the configuration values from the file.
    
2. Backup Settings: The script reads the necessary backup settings from the configuration file, such as the directory path to be backed up, the destination server, the backup type (`scp` or `rclone`), and the desired name for the backup archive file. Make sure to replace these values with your own in the script.
    
3. Creating the Archive: The script uses the `tar` command to create a compressed tar archive of the specified directory using the `tar -czf` command.
    
4. Copying the Archive: Depending on the backup type specified in the configuration file, the script either uses the `scp` command (for `scp` backup) or the `rclone` command (for `rclone` backup) to copy the archive to the destination server. If the copying process fails, it removes the local archive file.
    
5. Deleting the Local Archive: After successfully copying the archive to the server, the script deletes the local archive file to save disk space.
    

### **Running the Backup Script**

To run the backup script, follow these steps:

1. Save the script to a file named `backup.go` in a directory of your choice.
    
2. Set the `BACKUP_CONFIG_PATH` environment variable to the path of your configuration file. For example, you can run the following command in the terminal:
    
    ```go
    export BACKUP_CONFIG_PATH=/path/to/backup_config.yaml
    ```
    
3. Open a terminal and navigate to the directory containing the `backup.go` file.
    
4. Build and run the script using the `go run` command:
    
    ```go
    go run backup.go
    ```
    
5. The script will read the configuration file, create a backup archive of the specified directory, and copy it to the destination server using the specified backup type. The progress and any errors will be displayed in the terminal.
    
6. Once the backup process is complete, the local archive file will be deleted, and a success message will be displayed.
    

Congratulations! You have successfully automated the backup process using Go.

### **Conclusion**

We wrote a backup script that reads the backup settings from a configuration file, creates a compressed tar archive of a specified directory, and copies it to a destination server using either `scp` or `rclone`. By following the steps outlined in this guide, you can ensure the safety and availability of your important files through automated backups.

Git repository link can be found [here.](https://github.com/manitaggarwal/backup-script-in-go)