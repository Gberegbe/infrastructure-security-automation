# PHP Application Dockerfile with Apache and Healthcheck
# Base image: PHP 8.2 with Apache pre-configured

FROM php:8.2-apache

# Build argument for username (optional, for logging)
ARG USERNAME=default-user

# Install curl for healthcheck functionality
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Copy application source files
COPY src/ /var/www/html/

# Set proper ownership for web files
RUN chown -R www-data:www-data /var/www/html

# Configure Apache ServerName to suppress warning
RUN echo "ServerName localhost" > /etc/apache2/conf-available/servername.conf \
    && a2enconf servername

# Expose HTTP port
EXPOSE 80

# Docker healthcheck - verify application responds
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost/ || exit 1

# Startup command with optional logging
CMD ["/bin/sh", "-c", "echo \"Application started by ${USERNAME}\" && apache2-foreground"]
