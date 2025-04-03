# Docker Commands Cheatsheet

## Container Management
```
docker run <image>                                  # Run a container from an image
docker run -d <image>                               # Run a container in detached mode
docker run -it <image>                              # Run a container interactively with a terminal
docker run --name <name> <image>                    # Run a container with a custom name
docker run -p <host_port>:<container_port> <image>  # Map a host port to a container port
docker run -v <host_path>:<container_path> <image>  # Mount a host directory as a volume
docker run --rm <image>                             # Automatically remove the container after it stops
docker run --env <key>=<value> <image>              # Set environment variables in the container
docker run --network <network> <image>              # Connect a container to a specific network
docker start <container>                            # Start a stopped container
docker stop <container>                             # Stop a running container
docker restart <container>                          # Restart a container
docker pause <container>                            # Pause a running container
docker unpause <container>                          # Unpause a paused container
docker kill <container>                             # Forcefully stop a container
docker rm <container>                               # Remove a stopped container
docker rm -f <container>                            # Forcefully remove a running container
docker exec -it <container> <command>               # Execute a command in a running container
docker attach <container>                           # Attach to a running container
docker logs <container>                             # View logs of a container
docker logs -f <container>                          # Follow logs in real-time
docker ps                                           # List running containers
docker ps -a                                        # List all containers (running and stopped)
docker inspect <container>                          # Display detailed information about a container
docker top <container>                              # Display running processes in a container
docker stats <container>                            # Display live resource usage statistics
docker rename <old_name> <new_name>                 # Rename a container
docker update <container>                           # Update container configuration (e.g., memory, CPU)
docker commit <container> <image>                   # Create an image from a container
docker diff <container>                             # Inspect changes to files in a container
```

## Image Management
```
docker images                                       # List all images
docker pull <image>                                 # Download an image from a registry
docker push <image>                                 # Upload an image to a registry
docker build -t <tag> <path>                        # Build an image from a Dockerfile
docker rmi <image>                                  # Remove an image
docker tag <source> <target>                        # Tag an image
docker save -o <file>.tar <image>                   # Save an image to a tar file
docker load -i <file>.tar                           # Load an image from a tar file
docker history <image>                              # Show the history of an image
docker image prune                                  # Remove unused images
docker image inspect <image>                        # Inspect details of an image
docker image ls                                     # List all images (alternative to docker images)
docker image rm <image>                             # Remove an image (alternative to docker rmi)
docker image prune -a                               # Remove all unused images, including dangling ones
docker image pull <image>                           # Pull an image (alternative to docker pull)
docker image push <image>                           # Push an image (alternative to docker push)
```

## Networking
```
docker network ls                                   # List all networks
docker network create <network>                     # Create a new network
docker network inspect <network>                    # Inspect a network
docker network connect <network> <container>        # Connect a container to a network
docker network disconnect <network> <container>     # Disconnect a container from a network
docker network rm <network>                         # Remove a network
docker network prune                                # Remove unused networks
docker network create --driver bridge <network>     # Create a bridge network
docker network create --driver overlay <network>    # Create an overlay network (for Swarm)
```

## Volume Management
```
docker volume ls                                    # List all volumes
docker volume create <volume>                       # Create a new volume
docker volume inspect <volume>                      # Inspect a volume
docker volume rm <volume>                           # Remove a volume
docker volume prune                                 # Remove unused volumes
docker volume create --driver local <volume>        # Create a local volume
docker volume create --driver nfs <volume>          # Create an NFS volume
```

## Docker Compose
```
docker-compose up                                   # Start services defined in docker-compose.yml
docker-compose up -d                                # Start services in detached mode
docker-compose down                                 # Stop and remove services
docker-compose build                                # Build images for services
docker-compose logs                                 # View logs for services
docker-compose logs -f                              # Follow logs in real-time
docker-compose ps                                   # List running services
docker-compose exec <service> <command>             # Execute a command in a running service
docker-compose restart                              # Restart services
docker-compose pause                                # Pause services
docker-compose unpause                              # Unpause services
docker-compose config                               # Validate and view the Compose file
docker-compose pull                                 # Pull images for services
docker-compose push                                 # Push images for services
docker-compose scale <service>=<num>                # Scale a service to multiple instances
docker-compose rm                                   # Remove stopped services
docker-compose kill                                 # Forcefully stop services
docker-compose top                                  # Display running processes for services
```

## System and Maintenance
```
docker info                                         # Display system-wide information
docker version                                      # Display Docker version information
docker system df                                    # Show disk usage
docker system prune                                 # Remove unused data (containers, images, networks, volumes)
docker system prune -a                              # Remove all unused data, including dangling images
docker login                                        # Log in to a Docker registry
docker logout                                       # Log out from a Docker registry
docker search <term>                                # Search Docker Hub for images
docker events                                       # View real-time Docker events
docker checkpoint                                   # Manage checkpoints for containers (experimental)
docker node ls                                      # List nodes in a Swarm (Swarm mode)
docker service ls                                   # List services in a Swarm (Swarm mode)
docker stack deploy -c <compose_file> <stack_name>  # Deploy a stack in Swarm mode
docker stack rm <stack_name>                        # Remove a stack in Swarm mode
docker swarm init                                   # Initialize a Swarm
docker swarm join                                   # Join a Swarm as a worker or manager
docker swarm leave                                  # Leave a Swarm
docker plugin ls                                    # List installed plugins
docker plugin install <plugin>                      # Install a plugin
docker plugin rm <plugin>                           # Remove a plugin
```
