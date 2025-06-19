// Script to automatically update GitHub stats in README.md
import fs from 'fs';
import https from 'https';

const username = 'Shamimanowar';
const readmePath = './README.md';

// Function to get GitHub user stats
function getGitHubStats() {
  return new Promise((resolve, reject) => {
    const options = {
      hostname: 'api.github.com',
      path: `/users/${username}`,
      method: 'GET',
      headers: {
        'User-Agent': 'GitHub-Stats-Updater'
      }
    };

    const req = https.request(options, (res) => {
      let data = '';
      
      res.on('data', (chunk) => {
        data += chunk;
      });
      
      res.on('end', () => {
        try {
          const userData = JSON.parse(data);
          resolve({
            publicRepos: userData.public_repos,
            followers: userData.followers,
            following: userData.following
          });
        } catch (error) {
          reject(error);
        }
      });
    });

    req.on('error', (error) => {
      reject(error);
    });

    req.end();
  });
}

// Function to get current year contributions (approximate)
function getCurrentYearContributions() {
  return new Promise((resolve, reject) => {
    // This is a simplified approach - you would need to manually update this
    // or use GitHub GraphQL API for exact contribution counts
    const currentYear = new Date().getFullYear();
    
    // For now, we'll use a manual count that you can update
    // You can run this script and manually update the commit count
    const manualCommitCount = 1182; // Update this number manually
    
    resolve(manualCommitCount);
  });
}

// Function to update README.md
async function updateReadme() {
  try {
    const stats = await getGitHubStats();
    const commitCount = await getCurrentYearContributions();
    
    let readme = fs.readFileSync(readmePath, 'utf8');
    
    // Update commit count
    readme = readme.replace(
      /(\| ⏱️ \*\*Total Commits \(2025\):\*\* \| !\[GitHub commit activity\]\(https:\/\/img\.shields\.io\/badge\/Total%20Commits-)\d+(%2B-brightgreen\?style=social&logo=github\) \|)/,
      `$1${commitCount}$2`
    );
    
    // Update repositories count
    readme = readme.replace(
      /(\| 🚌 \*\*Repositories:\*\* \| !\[GitHub repos\]\(https:\/\/img\.shields\.io\/badge\/dynamic\/json\?url=https%3A%2F%2Fapi\.github\.com%2Fusers%2F)\w+(&query=%24\.public_repos&style=social&logo=github&label=Public%20Repos&color=blue\) \|)/,
      `$1${username}$2`
    );
    
    fs.writeFileSync(readmePath, readme);
    
    console.log('✅ README.md updated successfully!');
    console.log(`📊 Stats updated:`);
    console.log(`   - Total Commits (2025): ${commitCount}`);
    console.log(`   - Public Repositories: ${stats.publicRepos}`);
    console.log(`   - Followers: ${stats.followers}`);
    
  } catch (error) {
    console.error('❌ Error updating README:', error);
  }
}

// Run the update
updateReadme();
