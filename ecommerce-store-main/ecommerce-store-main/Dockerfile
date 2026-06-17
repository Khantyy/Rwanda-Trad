# Use a lightweight Nginx image to serve static files
FROM nginx:alpine

# Copy the application files to the Nginx web root
COPY . /usr/share/nginx/html

# Expose port 80
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]