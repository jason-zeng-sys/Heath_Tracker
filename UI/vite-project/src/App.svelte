<script>
  import { onMount } from "svelte";

  let viewingPrevious = false;
  let submissionMessage = "";
  let editing = false;

  let userColor = '#ffffff'; // default background color

  // Mock previous entries
  let mockEntries = [
    {
      date: "2024-09-17",
      userName: "Ming Zeng",
      totalCalories: 1800,
      gymDuration: 40,
      waterIntake: 7,
      mood: "Happy",
      exerciseType: "Yoga",
      foodsEaten: [
        { name: "Vegetables", calories: 50 },
        { name: "Fruits", calories: 80 },
        { name: "KFC", calories: 700 }
      ],
    },
    {
      date: "2024-09-18",
      userName: "Ming Zeng",
      totalCalories: 1800,
      gymDuration: 30,
      waterIntake: 7,
      mood: "Happy",
      exerciseType: "Running",
      foodsEaten: [
        { name: "Vegetables", calories: 50 },
        { name: "Fruits", calories: 80 },
        { name: "KFC", calories: 700 }
      ],
    },
  ];

  let userName = "Ming Zeng";
  let foods = [
    { name: "Vegetables", calories: 50 },
    { name: "Fruits", calories: 80 },
    { name: "KFC", calories: 700 },
    { name: "Dairy", calories: 150 },
    { name: "Chipotle", calories: 500 },
    { name: "Smoothie King", calories: 300 },
    { name: "Chick-fil-A", calories: 600 },
    { name: "Wingstop", calories: 450 },
    { name: "Starbucks", calories: 250 },
    { name: "Panda Express", calories: 800 },
    { name: "Snacks", calories: 200 },
    { name: "Beer", calories: 150 }
  ];
  let foodsEatenToday = Array(foods.length).fill(false);
  let totalCaloriesToday = 0;
  let gymDuration = 0;
  let waterIntake = 0;
  let todayMood = "";
  let exerciseType = "None";
  let currentDate = new Date();
  let currentEntryIndex = 0;

  let gymDurationGoal = 30;
  let waterIntakeGoal = 8;
  let calorieGoal = 2000;

  let averageCalories = 0;
  let averageGymDuration = 0;
  let averageWaterIntake = 0;

  function calculateAverages() {
    let totalEntries = mockEntries.length;
    averageCalories = mockEntries.reduce((acc, entry) => acc + entry.totalCalories, 0) / totalEntries;
    averageGymDuration = mockEntries.reduce((acc, entry) => acc + entry.gymDuration, 0) / totalEntries;
    averageWaterIntake = mockEntries.reduce((acc, entry) => acc + entry.waterIntake, 0) / totalEntries;
  }

  calculateAverages();

  function updateCaloriesToday() {
    totalCaloriesToday = foods.reduce((sum, food, index) => foodsEatenToday[index] ? sum + food.calories : sum, 0);
  }

  function handleSubmit() {
    const newEntry = {
      date: currentDate.toLocaleDateString(),
      userName: userName,
      totalCalories: totalCaloriesToday,
      gymDuration: gymDuration,
      waterIntake: waterIntake,
      mood: todayMood,
      exerciseType: exerciseType,
      foodsEaten: foods.filter((_, index) => foodsEatenToday[index]).map(food => ({
        name: food.name,
        calories: food.calories
      })),
    };

    mockEntries.push(newEntry);
    calculateAverages(); // Recalculate averages with the new entry
    resetEntryForm();
    submissionMessage = "Today's entry has been submitted!";
    setTimeout(() => submissionMessage = "", 3000);
  }

  function resetEntryForm() {
    foodsEatenToday.fill(false);
    totalCaloriesToday = 0;
    gymDuration = 0;
    waterIntake = 0;
    todayMood = "";
    exerciseType = "None";
  }

  function previousEntry() {
    currentEntryIndex = Math.max(currentEntryIndex - 1, 0);
  }

  function nextEntry() {
    currentEntryIndex = Math.min(currentEntryIndex + 1, mockEntries.length - 1);
  }

  function toggleEdit() {
    editing = !editing;
    if (editing && mockEntries.length > 0) {
      loadEntryForEditing();
    }
  }

  function loadEntryForEditing() {
    let entry = mockEntries[currentEntryIndex];
    userName = entry.userName;
    totalCaloriesToday = entry.totalCalories;
    gymDuration = entry.gymDuration;
    waterIntake = entry.waterIntake;
    todayMood = entry.mood;
    exerciseType = entry.exerciseType;
    foodsEatenToday = foods.map(food => entry.foodsEaten.some(e => e.name === food.name));
  }

  function saveEditedEntry() {
    let editedEntry = {
      date: mockEntries[currentEntryIndex].date,
      userName: userName,
      totalCalories: totalCaloriesToday,
      gymDuration: gymDuration,
      waterIntake: waterIntake,
      mood: todayMood,
      exerciseType: exerciseType,
      foodsEaten: foods.filter((_, index) => foodsEatenToday[index]).map(food => ({
        name: food.name,
        calories: food.calories
      })),
    };

    mockEntries[currentEntryIndex] = editedEntry;
    editing = false;

    submissionMessage = "Entry updated successfully!";
    setTimeout(() => submissionMessage = "", 7000);
  }

  onMount(() => {
    const interval = setInterval(() => currentDate = new Date(), 1000);
    return () => clearInterval(interval);
  });

  function applyBackgroundColor(event) {
    userColor = event.target.value;
  }
</script>


<main style="background-color: {userColor}">
  <div class="color-picker">
    <input type="color" on:input={applyBackgroundColor} />
    <button on:click={applyBackgroundColor}>Customize Colors</button>
  </div>
  
 <!-- Display submission or edit message -->
 {#if submissionMessage}
 <p class="message">{submissionMessage}</p>
{/if}


  <div class="grid-container">
    <!-- Average Log Overview Section -->
    <section class="average-overview">
      <h2>Your Average Log Overview</h2>
      <p><strong>Average Calories:</strong> {averageCalories.toFixed(2)} cal</p>
      <p><strong>Average Gym Duration:</strong> {averageGymDuration.toFixed(2)} minutes</p>
      <p><strong>Average Water Intake:</strong> {averageWaterIntake.toFixed(2)} glasses</p>

      <!-- Comparing averages with goals -->
      <h3>Comparison with Goals:</h3>
      <p><strong>Calorie Goal:</strong> {calorieGoal} cal</p>
      <p><strong>Gym Duration Goal:</strong> {gymDurationGoal} minutes</p>
      <p><strong>Water Intake Goal:</strong> {waterIntakeGoal} glasses</p>

      <h4>How are you doing?</h4>
      <p>{#if averageCalories > calorieGoal} You're exceeding your calorie goal! 
        {:else if averageCalories < calorieGoal} You're under your calorie goal. Keep it up! 
        {:else} You're hitting your calorie goal perfectly! {/if}</p>

      <p>{#if averageGymDuration > gymDurationGoal} You're working out more than your goal! 
        {:else if averageGymDuration < gymDurationGoal} You might want to increase your workout duration. 
        {:else} You're hitting your workout goal! {/if}</p>

      <p>{#if averageWaterIntake > waterIntakeGoal} Great job, you're drinking more than enough water! 
        {:else if averageWaterIntake < waterIntakeGoal} Drink a bit more water to reach your goal! 
        {:else} You're staying hydrated perfectly! {/if}</p>
    </section>

    <!-- User Overview Section -->
    <section class="user-overview">
      <h2>User Overview</h2>
      <p>Name: {mockEntries[currentEntryIndex].userName}</p>
      <p>Current Date/Time: {currentDate.toLocaleString()}</p>
      <p>Days since starting: {((currentDate - new Date("2024-01-01")) / (1000 * 60 * 60 * 24)).toFixed(2)} days</p>

      <div>
        <button on:click={previousEntry} disabled={currentEntryIndex === 0}>Previous Entry</button>
        <button on:click={nextEntry} disabled={currentEntryIndex === mockEntries.length - 1}>Next Entry</button>
      </div>

      <h3>Entry for {mockEntries[currentEntryIndex].date}</h3>

      <p><strong>Total Calories:</strong> {mockEntries[currentEntryIndex].totalCalories} cal</p>
      <p><strong>Gym Duration:</strong> {mockEntries[currentEntryIndex].gymDuration} minutes</p>
      <p><strong>Water Intake:</strong> {mockEntries[currentEntryIndex].waterIntake} glasses</p>
      <p><strong>Mood:</strong> {mockEntries[currentEntryIndex].mood}</p>
      <p><strong>Exercise Type:</strong> {mockEntries[currentEntryIndex].exerciseType}</p>

      <h4>Foods Eaten:</h4>
      <ul>
        {#each mockEntries[currentEntryIndex].foodsEaten as food}
          <li>{food.name} ({food.calories} cal)</li>
        {/each}
      </ul>

      {#if editing}
        <h3>Editing Entry for {mockEntries[currentEntryIndex].date}</h3>

        <label for="userName">Edit Name:</label>
        <input type="text" id="userName" bind:value={userName} />

        <h3>What did you eat today? (Select all that apply)</h3>
        {#each foods as food, index}
          <label>
            <input type="checkbox" bind:checked={foodsEatenToday[index]} on:change={updateCaloriesToday} />
            {food.name} ({food.calories} cal)
          </label>
        {/each}
        <p>Total Calories: {totalCaloriesToday} cal</p>

        <h3>Did you go to the gym? If yes, for how long (in minutes)?</h3>
        <input type="number" bind:value={gymDuration} min="0" />

        <h3>How many glasses of water did you drink today?</h3>
        <input type="range" min="0" max="10" bind:value={waterIntake} />
        <p>{waterIntake} glasses</p>

        <h3>Describe your mood today:</h3>
        <textarea bind:value={todayMood}></textarea>

        <button on:click={saveEditedEntry}>Save</button>
      {/if}

      {#if !editing}
        <button on:click={toggleEdit}>Edit</button>
      {/if}
    </section>

    <!-- Add Today's Entry Section -->
    <section class="todays-entry">
      <h2>Enter Today's Activities</h2>
      <label for="userName">Enter your name:</label>
      <input type="text" id="userName" bind:value={userName} />

      <h3>What did you eat today? (Select all that apply)</h3>
      {#each foods as food, index}
        <label>
          <input type="checkbox" bind:checked={foodsEatenToday[index]} on:change={updateCaloriesToday} />
          {food.name} ({food.calories} cal)
        </label>
      {/each}
      
      <p>Total Calories: {totalCaloriesToday} cal</p>

      <h3>Did you go to the gym? If yes, for how long (in minutes)?</h3>
      <input type="number" bind:value={gymDuration} min="0" placeholder="Enter minutes" />

      <h3>How many glasses of water did you drink today?</h3>
      <input type="range" min="0" max="10" bind:value={waterIntake} />
      <p>{waterIntake} glasses</p>

      <h3>Describe your mood today:</h3>
      <textarea bind:value={todayMood}></textarea>

      <h3>What type of exercise did you do today?</h3>
      <select bind:value={exerciseType}>
        <option value="None">None</option>
        <option value="Walking">Walking</option>
        <option value="Running">Running</option>
        <option value="Cycling">Cycling</option>
        <option value="Yoga">Yoga</option>
        <option value="Other">Other</option>
      </select>

      <!-- Goal input fields only for new entries -->
      <h3>Set Goals:</h3>
      <label for="gymDurationGoal">Gym Duration Goal (minutes):</label>
      <input type="number" id="gymDurationGoal" bind:value={gymDurationGoal} min="0" />

      <label for="waterIntakeGoal">Water Intake Goal (glasses):</label>
      <input type="number" id="waterIntakeGoal" bind:value={waterIntakeGoal} min="0" />

      <label for="calorieGoal">Calorie Goal:</label>
      <input type="number" id="calorieGoal" bind:value={calorieGoal} min="0" />

      <button on:click={handleSubmit}>Submit Today's Log</button>
    </section>
  </div>

  
</main>

<style>
  :root {
    --background-color: #ffffff;
    --text-color: #000000;
  }

  main {
  padding: 20px;
  text-align: center;
  min-height: 100vh; /* Full height of the viewport */
  background-color: var(--userColor, white); /* Use the color selected by user */
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* Two equal columns */
  grid-gap: 20px; /* Space between grid items */
  padding: 20px;
  max-width: 1200px; /* Set a max width to center the grid */
  margin: 0 auto; /* Center the container */
}

.average-overview, .user-overview, .todays-entry {
  border: 2px solid #ccc;
  padding: 20px;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.average-overview {
  grid-column: span 2; /* Span both columns for the average overview */
}

.color-picker {
  position: absolute;
  top: 20px;
  right: 20px;
}

button {
  margin: 10px;
  padding: 10px 20px;
  font-size: 16px;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  font-size: 16px;
  margin-bottom: 8px;
}

.message {
  color: green;
  font-size: 18px;
  margin: 10px 0;
}

</style>
