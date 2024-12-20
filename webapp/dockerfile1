# Use the official Nginx image to serve static files
FROM nginx:alpine

# Copy the build files (dist folder) from the host to the Nginx server
COPY ./dist /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]

