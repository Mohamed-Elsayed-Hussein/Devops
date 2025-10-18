# -----First stage ------
FROM python:3.11-slim As builder

WORKDIR /app

# Install system build tools required by some Python packages
# RUN apt-get update && apt-get install -y --no-install-recommends build-essential
COPY requirement.txt /app/

# Install Python dependencies into a dedicated folder (/app/requirement)
RUN pip install -r requirement.txt --target=/app/requirement


# -----Final stage ------
FROM python:3.11-slim 

WORKDIR /app

# Copy installed site-packages from the builder stage into the runtime's site-packages
COPY --from=builder  /app/requirement /usr/local/lib/python3.11/site-packages

# Copy the application code into the image
COPY ./app.py /app/ 

# Create a non-root user for improved security and switch to it
RUN useradd -m appuser
RUN chown -R appuser:appuser /app


USER appuser




# Document the port the application listens on & and *open it on the container* 
EXPOSE 5000

# Default command to run the application
CMD ["python", "app.py"]