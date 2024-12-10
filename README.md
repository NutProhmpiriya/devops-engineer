# Linux Command Practice Environment

This Docker Compose setup provides a safe environment for practicing Linux commands.

## Getting Started

1. Start the container:
```bash
docker-compose up -d
```

2. Enter the container:
```bash
docker-compose exec linux-practice bash
```

3. Now you can practice Linux commands!

Some basic commands to try:
- `ls` - List files and directories
- `pwd` - Print working directory
- `mkdir` - Create directory
- `touch` - Create empty file
- `cat` - View file contents
- `cp` - Copy files
- `mv` - Move/rename files
- `rm` - Remove files
- `grep` - Search text
- `ps` - Show processes

4. To exit the container:
```bash
exit
```

5. To stop the container:
```bash
docker-compose down
```

Note: Files in the `practice-data` directory will persist even after container restart.
