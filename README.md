# medium-articles-fetcher
convert below into markdown for github Build API To Fetch Medium Articles and convert it into a Docker Image Make a directory on called medium-fetch-api and change directory to it. mkdir medium-fetch-api cd medium-fetch-api install npm with the below command. sudo apt install npm make a directory called public and change directory into it. mkdir public cd public Create a index.html file in public directory with below html code. <!DOCTYPE html> <html lang="en"> <head> <meta charset="UTF-8"> <meta name="viewport" content="width=device-width, initial-scale=1.0"> <title>My Projects</title> <style> body { font-family: Arial, sans-serif; } h2 { margin: 20px 0; } section { margin-bottom: 20px; } a { color: blue; text-decoration: underline; } </style> </head> <body> <h2>Medium Articles</h2> <div id="medium-articles"></div> <script> fetch('/medium-articles') .then(response => response.text()) .then(str => (new window.DOMParser()).parseFromString(str, "text/xml")) .then(data => { const items = data.querySelectorAll("item"); const mediumArticlesDiv = document.getElementById('medium-articles'); items.forEach((el) => { const title = el.querySelector("title").textContent; const link = el.querySelector("link").textContent; const description = el.querySelector("description").textContent; const article = document.createElement('section'); article.innerHTML = \` <h3>${title}</h3> <p>${description}</p> <a href="${link}" target="\_blank">Read More</a> \`; mediumArticlesDiv.appendChild(article); }); }) .catch(err => console.error('Error fetching Medium articles:', err)); </script> </body> </html> Now create a file called server.mjs into medium-fetch-api directory with below command cd .. sudo nano server.mjs And copy the below node.js code into it. import express from 'express'; import fetch from 'node-fetch'; import cors from 'cors'; import { DOMParser } from 'xmldom'; // Import xmldom to handle XML parsing const app = express(); const PORT = process.env.PORT || 5000; // Use CORS middleware app.use(cors()); app.use(express.static('public')); // Serve static files from the 'public' directory // API endpoint to fetch Medium articles app.get('/medium-articles', async (req, res) => { try { const response = await fetch('https://medium.com/feed/@joysonfernandes'); const body = await response.text(); // Parse the XML response const xmlData = new DOMParser().parseFromString(body, 'text/xml'); const items = xmlData.getElementsByTagName('item'); // Get all <item> elements // Build an array of articles const articles = Array.from(items).map(item => { return { title: item.getElementsByTagName('title')\[0\]?.textContent || 'No Title Available', link: item.getElementsByTagName('link')\[0\]?.textContent || '#', description: item.getElementsByTagName('description')\[0\]?.textContent || 'No Description Available', content: item.getElementsByTagName('content:encoded')\[0\]?.textContent || 'No Content Available', }; }); res.json(articles); // Send back the articles as JSON } catch (error) { console.error('Error fetching Medium articles:', error.message); res.status(500).send('Error fetching Medium articles'); } }); app.listen(PORT, () => { console.log(\`Server is running on port ${PORT}\`); }); install all the packages with the below commands. npm install express node-fetch npm install cors npm install xmldom Run below command to start the server node server.mjs Build a Docker Container Create a Dockerfile in root of medium-fetch-api directory with below code. # Use a Node.js base image FROM node:16 # Set the working directory in the container WORKDIR /app # Copy package.json and package-lock.json (if available) into the container COPY package\*.json ./ # Install the dependencies RUN npm install # Copy the rest of your application code COPY . . # Expose the port the app runs on EXPOSE 5000 # Command to run the app CMD \["node", "server.mjs"\] Build the Docker Image: In your terminal, navigate to your project directory and run to build the docker image Please: Make sure docker is installed docker build -t medium-fetch-api . Run the Docker Container: After successfully building the image, run the container using: docker run -d --name my\_container -p 5000:5000 medium-fetch-api This command maps port 5000 on your host to port 5000 in the container. Access Your Application: Open your web browser and go to http://localhost:5000 to see your application in action. Pushing image to Docker Hub Rename the image if required Find the Existing Image ID or Name: First, list your current Docker images to find the image you want to rename. You can do this by running: docker images Tag the Image: Use the docker tag command to assign a new name to your image. The syntax is: docker tag <existing\_image\_name\_or\_id> <new\_image\_name> For example, if the existing image is old\_image\_name:latest and you want to rename it to new\_image\_name:latest, you would run: docker tag old\_image\_name:latest new\_image\_name:latest Verify the Change: Run docker images again to see the new image listed alongside the old one. The old image will remain until you explicitly remove it. Removing the Old Image If you want to completely remove the old image after renaming it, you can do the following: Remove the Old Image: After tagging the new image, remove the old image with the following command: docker rmi old\_image\_name:latest Verify Removal: Again, run docker images to ensure that the old image has been removed. Log In to Docker Hub Open your terminal and log in to your Docker Hub account by running: docker login You'll be prompted to enter your Docker Hub username and password. Tag Your Image You need to tag your image in a specific format before pushing it to Docker Hub. The format is: docker.io/<username>/<repository>:<tag> For example, if your Docker Hub username is yourusername, the repository name is yourrepository, and the tag is latest, you would do: docker tag your\_image\_name yourusername/yourrepository:latest Push the Image Use the following command to push the image to Docker Hub: docker push yourusername/yourrepository:latest
