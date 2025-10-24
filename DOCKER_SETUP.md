# Docker Setup Guide for Strapi with Supabase

This guide helps you run your Strapi application in Docker using Supabase as your database.

## Prerequisites

- Docker installed on your system
- Docker Compose installed
- Supabase project created and credentials ready

## Quick Start

### 1. Set Up Environment Variables

Copy the `.env.example` file to `.env.local` and fill in your Supabase credentials:

```bash
cp .env.example .env.local
```

Edit `.env.local` with your Supabase connection details:

```env
# Supabase Database Configuration
DATABASE_HOST=your-project-ref.supabase.co
DATABASE_PORT=5432
DATABASE_NAME=your_database_name
DATABASE_USERNAME=your_database_user
DATABASE_PASSWORD=your_secure_password
DATABASE_SSL_REJECT_UNAUTHORIZED=false

# Strapi Configuration (generate secure random strings)
ADMIN_JWT_SECRET=your_secure_admin_jwt_secret
JWT_SECRET=your_secure_jwt_secret
API_TOKEN_SALT=your_secure_api_token_salt
APP_KEYS=key1,key2,key3,key4
```

**Finding your Supabase credentials:**
1. Go to [Supabase Console](https://supabase.com/dashboard)
2. Select your project
3. Click "Connect"
4. Choose "URI" tab
5. Copy the connection string parameters

### 2. Build and Run with Docker Compose

```bash
# Build the Docker image
docker-compose build

# Start the application
docker-compose up -d

# View logs
docker-compose logs -f strapi
```

The Strapi admin panel will be available at `http://localhost:1337/admin`

### 3. Stop the Application

```bash
docker-compose down
```

## Building the Docker Image Manually

If you prefer to build the image without Docker Compose:

```bash
docker build -t strapi-app:latest .

docker run -d \
  --name strapi \
  -p 1337:1337 \
  -e DATABASE_HOST=your-project.supabase.co \
  -e DATABASE_PORT=5432 \
  -e DATABASE_NAME=postgres \
  -e DATABASE_USERNAME=postgres \
  -e DATABASE_PASSWORD=your_password \
  -e ADMIN_JWT_SECRET=your_secret \
  -e JWT_SECRET=your_secret \
  -e API_TOKEN_SALT=your_salt \
  -e APP_KEYS=key1,key2,key3,key4 \
  -e NODE_ENV=production \
  strapi-app:latest
```

## Directory Structure

```
/app/
├── dist/              # Compiled TypeScript
├── .cache/            # Strapi cache
├── config/            # Configuration files
├── public/
│   └── uploads/       # User uploads (persisted as volume)
├── src/               # Source code
└── types/             # Type definitions
```

## Volumes

The `docker-compose.yml` mounts:
- `./public/uploads:/app/public/uploads` - Persists uploaded files on the host machine

To add more volumes (e.g., for database backups), modify the `volumes` section in `docker-compose.yml`.

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_HOST` | Supabase host | Yes |
| `DATABASE_PORT` | Supabase port (usually 5432) | Yes |
| `DATABASE_NAME` | Database name (usually postgres) | Yes |
| `DATABASE_USERNAME` | Database user | Yes |
| `DATABASE_PASSWORD` | Database password | Yes |
| `DATABASE_SSL_REJECT_UNAUTHORIZED` | SSL verification (false for Supabase) | No |
| `ADMIN_JWT_SECRET` | Secret for admin panel JWT | Yes |
| `JWT_SECRET` | Secret for API JWT | Yes |
| `API_TOKEN_SALT` | Salt for API tokens | Yes |
| `APP_KEYS` | Encryption keys (comma-separated) | Yes |
| `NODE_ENV` | Environment (production/development) | No (defaults to production) |

## Troubleshooting

### Container won't start
```bash
docker-compose logs strapi
```

Check for database connection errors. Verify your Supabase credentials are correct.

### Permission denied for uploads directory
```bash
chmod 755 public/uploads
```

### Clear everything and start fresh
```bash
docker-compose down -v
docker rmi strapi-app:latest
docker-compose build --no-cache
docker-compose up -d
```

### Database connection timeout
- Verify Supabase IP whitelist includes your Docker host
- Check that `DATABASE_SSL_REJECT_UNAUTHORIZED=false` is set
- Ensure firewall allows outbound connections on port 5432

## Production Deployment

For production, consider:

1. **Use a `.env` file** (not `.env.local`) with strong, randomly generated secrets
2. **Enable SSL/TLS** for your database connection
3. **Use health checks** (included in docker-compose.yml)
4. **Set restart policy** (included: `unless-stopped`)
5. **Implement backup strategies** for the uploads directory
6. **Monitor container logs** and resource usage
7. **Use Docker secrets** in Docker Swarm or Kubernetes for sensitive data

## Useful Commands

```bash
# View running containers
docker ps

# View container logs
docker logs strapi-app

# Follow logs in real-time
docker logs -f strapi-app

# Execute command in running container
docker exec -it strapi-app sh

# Rebuild image after code changes
docker-compose build --no-cache
docker-compose up -d

# Remove unused images and volumes
docker system prune -a --volumes
```

## Next Steps

1. Run migrations if needed
2. Seed your database using `pnpm run seed:example`
3. Access admin panel at `http://localhost:1337/admin`
4. Create your admin user
5. Configure content types and permissions

For more information, see [Strapi Docker Documentation](https://docs.strapi.io/dev-docs/deployment/docker)
