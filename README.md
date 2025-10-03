import jsonfile from "jsonfile";
import moment from "moment";
import simpleGit from "simple-git";
import random from "random";

const path = "./data.json";

// Configuration for realistic patterns
const CONFIG = {
  startDate: '2023-01-01',
  endDate: '2025-10-03',
  weekdayProbability: 0.7,    // 70% chance of commits on weekdays
  weekendProbability: 0.3,    // 30% chance of commits on weekends
  vacationProbability: 0.1,   // 10% chance of vacation periods (no commits for 3-7 days)
  maxCommitsPerDay: 5,        // Maximum commits per active day
  minCommitsPerDay: 1,        // Minimum commits per active day
};

const createRealisticData = (date) => {
  const projectTypes = [
    'Updated documentation',
    'Fixed bug in authentication',
    'Added new feature',
    'Refactored code structure',
    'Updated dependencies',
    'Improved performance',
    'Added unit tests',
    'Fixed styling issues',
    'Updated configuration',
    'Merged pull request'
  ];
  
  return {
    date: date,
    activity: projectTypes[random.int(0, projectTypes.length - 1)],
    commits: random.int(CONFIG.minCommitsPerDay, CONFIG.maxCommitsPerDay)
  };
};

const shouldCommitOnDate = (momentDate) => {
  const isWeekend = momentDate.day() === 0 || momentDate.day() === 6;
  const probability = isWeekend ? CONFIG.weekendProbability : CONFIG.weekdayProbability;
  
  // Add some vacation periods (longer gaps)
  if (Math.random() < CONFIG.vacationProbability / 30) { // Roughly once per month
    return false;
  }
  
  return Math.random() < probability;
};

const generateRealisticCommits = async () => {
  console.log('🚀 Starting Sapkoto Tech Contribution Generator...');
  console.log('⚠️  WARNING: This may violate GitHub Terms of Service');
  console.log('📅 Generating commits from', CONFIG.startDate, 'to', CONFIG.endDate);
  
  const startDate = moment(CONFIG.startDate);
  const endDate = moment(CONFIG.endDate);
  const git = simpleGit();
  
  let commitCount = 0;
  let currentDate = startDate.clone();
  
  while (currentDate.isSameOrBefore(endDate)) {
    if (shouldCommitOnDate(currentDate)) {
      const dateStr = currentDate.format();
      const data = createRealisticData(dateStr);
      
      // Create multiple commits for the day if needed
      for (let i = 0; i < data.commits; i++) {
        const commitTime = currentDate.clone()
          .add(random.int(8, 22), 'hours')  // Commits between 8 AM and 10 PM
          .add(random.int(0, 59), 'minutes')
          .format();
        
        const commitData = {
          date: commitTime,
          activity: data.activity,
          commitNumber: i + 1,
          totalForDay: data.commits
        };
        
        try {
          await new Promise((resolve, reject) => {
            jsonfile.writeFile(path, commitData, (err) => {
              if (err) return reject(err);
              
              git.add([path])
                .commit(`${data.activity} - ${currentDate.format('YYYY-MM-DD')}`, 
                       { '--date': commitTime })
                .then(() => {
                  commitCount++;
                  console.log(`✅ ${commitCount}: ${commitTime.substring(0, 10)} - ${data.activity}`);
                  resolve();
                })
                .catch(reject);
            });
          });
          
          // Small delay to avoid overwhelming the system
          await new Promise(resolve => setTimeout(resolve, 100));
          
        } catch (error) {
          console.error(`❌ Error creating commit for ${dateStr}:`, error.message);
        }
      }
    }
    
    currentDate.add(1, 'day');
  }
  
  console.log(`\n🎉 Generated ${commitCount} commits successfully!`);
  console.log('📤 Pushing to remote repository...');
  
  try {
    await git.push();
    console.log('✅ Successfully pushed all commits to remote!');
    console.log('\n📊 Your GitHub contribution graph should now show activity from 2023 to 2025');
    console.log('⚠️  Remember: This is for educational purposes only');
  } catch (error) {
    console.error('❌ Error pushing commits:', error.message);
    console.log('💡 You may need to push manually: git push origin main');
  }
};

// Execute the realistic commit generation
generateRealisticCommits().catch(console.error);
