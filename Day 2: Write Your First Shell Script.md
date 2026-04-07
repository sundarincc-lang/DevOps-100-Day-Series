🌱 Day 1 – Write Your First Shell Script
📌 Goal: Automate routine tasks with basic shell scripting.

Activities:

Install a text editor (nano, vim, or gedit)
Learn shell script basics: variables, loops, conditionals
Write a script to list files in a directory and output to a file
Make the script executable with chmod +x script.sh
Execute and debug your first script



🐚 Shell Scripting Basics
This repository is a beginner‑friendly guide to shell scripting. It covers variables, loops, conditionals, and walks through writing, executing, and debugging your first script.
📘 Topics Covered
- Variables and how to use them
- Loops (for, while)
- Conditionals (if, else, elif)
- Writing a simple script
- Making scripts executable
- Debugging basics


📝 Example Script
#!/bin/bash

# Script to list files in a directory and save output to a file

# Define variables
DIR="./"          # directory to list
OUTPUT="files.txt" # output file

# Check if directory exists
if [ -d "$DIR" ]; then
    echo "Listing files in $DIR..."
    ls "$DIR" > "$OUTPUT"
    echo "File list saved to $OUTPUT"
else
    echo "Directory $DIR does not exist."
fi


⚙️ Steps to Run
- Save the script as script.sh.
- Make it executable:
chmod +x script.sh
- Run the script:
./script.sh


🐞 Debugging
Run with bash -x script.sh to see each command executed step‑by‑step.




📚 Resources
- GNU Bash Manual (gnu.org in Bing)
- Shell Scripting Tutorial
- Advanced Bash-Scripting Guide (tldp.org in Bing)


