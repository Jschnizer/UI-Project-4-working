<script>
    let giftsGiven = 1000;
    let totalBudget = 0;
    let dropOffLocationsList = new Set();
    let deliveredItems = [];

    let names = [
        "Oliver",
        "Emma",
        "Liam",
        "Sophia",
        "Noah",
        "Ava",
        "Elijah",
        "Isabella",
        "James",
        "Mia",
    ];

    let giftIdeas = [
        "LEGO building set",
        "stuffed animal",
        "remote-controlled car",
        "art supplies kit",
        "science experiment kit",
        "dollhouse",
        "puzzle set",
        "action figure",
        "storybook collection",
        "soccer ball",
    ];

    let dropOffLocations = [
        "123 Chapel Lane",
        "456 Schoolhouse Drive",
        "789 Church Road",
        "101 Candy Cane Street",
        "202 Snowday Avenue",
        "303 Winter Wonderland Blvd",
        "404 North Pole Circle",
        "505 Jingle Bell Way",
        "606 Frosty Lane",
        "707 Sleigh Ride Terrace",
    ];

    let leftColumnTiles = Array(10)
        .fill("")
        .map((_, index) => ({
            id: index + 1,
            name: names.at(index),
            age: Math.floor(Math.random() * (12 - 3 + 1)) + 3, // Random age between 3 and 12
            location: dropOffLocations.at(index),
            request: giftIdeas.at(index),
            budget: Math.floor(Math.random() * ((100 - 20) / 5 + 1)) * 5 + 20, // Random budget between $20 and $200
        }));

    let rightColumnTiles = []; // Initially empty

    function selectTile(index) {
        const tile = leftColumnTiles.splice(index, 1)[0]; // Remove tile from left column
        rightColumnTiles = [...rightColumnTiles, tile]; // Add tile to right column
        leftColumnTiles = [...leftColumnTiles];

        totalBudget += tile.budget;
        dropOffLocationsList.add(tile.location);

        updateMainContent();
    }

    function deliverTile(index) {
        rightColumnTiles.splice(index, 1); // Remove tile from right column
        rightColumnTiles = [...rightColumnTiles];

        deliveredItems = [
            ...deliveredItems,
            { ...tile, deliveredId: Date.now() },
        ];

        updateMainContent();
    }

    function markAsDelivered(index) {
        const tile = rightColumnTiles.splice(index, 1)[0]; // Remove the tile from the right column
        rightColumnTiles = [...rightColumnTiles];

        // Add the tile to the delivered items list
        deliveredItems = [
            ...deliveredItems,
            { ...tile, deliveredId: Date.now() }, // Add a unique ID for undo functionality
        ];

        // Deduct from total budget and drop-off location
        totalBudget -= tile.budget;
        dropOffLocationsList.delete(tile.location); // Remove drop-off location

        updateMainContent();
    }

    function undoDelivery(deliveredId) {
        // Find the item in the deliveredItems list
        const itemIndex = deliveredItems.findIndex(
            (item) => item.deliveredId === deliveredId,
        );

        if (itemIndex !== -1) {
            const item = deliveredItems.splice(itemIndex, 1)[0];
            delete item.deliveredId;

            rightColumnTiles = [...rightColumnTiles, item];

            // Re-add budget and location to totals
            totalBudget += item.budget; // Add back the budget
            dropOffLocationsList.add(item.location); // Re-add drop-off location

            // Trigger reactivity for deliveredItems and rightColumnTiles
            deliveredItems = [...deliveredItems];
            rightColumnTiles = [...rightColumnTiles];

            updateMainContent();
        }
    }
    function cancelTile(index) {
        const tile = rightColumnTiles.splice(index, 1)[0]; // Remove tile from right column
        rightColumnTiles = [...rightColumnTiles];

        // Add tile back to left column
        leftColumnTiles = [...leftColumnTiles, tile];

        // Deduct from total budget and drop-off location
        totalBudget -= tile.budget;
        dropOffLocationsList.delete(tile.location);

        updateMainContent();
    }

    function updateMainContent() {
        const budgetCounter = document.querySelector("#total-budget");
        const locationList = document.querySelector("#dropoff-locations");
        const deliveredList = document.querySelector("#delivered-list");

        // Update budget counter
        budgetCounter.textContent = `$${totalBudget}`;

        // Update dropoff locations list
        locationList.innerHTML = "";
        dropOffLocationsList.forEach((location) => {
            const li = document.createElement("li");
            li.textContent = location;
            locationList.appendChild(li);
        });

        // Update delivered items list
        deliveredList.innerHTML = "";
        deliveredItems.forEach((item) => {
            const row = document.createElement("div");
            row.className = "delivered-row";
            row.innerHTML = `
                <span>${item.name} - ${item.request}</span>
                <button class="undo-button" onclick="undoDelivery(${item.deliveredId})">Undo</button>
            `;
            deliveredList.appendChild(row);
        });
    }
</script>

<div class="top-bar">
    <div class="top-bar-labels">
        <div class="label left-label">Children in Need</div>
        <div class="gift-counter-container">
            <div class="gift-counter">Help Those in Need this Christmas</div>
        </div>
        <div class="label right-label">My Gifts</div>
    </div>
</div>

<div class="layout">
    <!-- Scrollable Column (Left) -->
    <div class="scrollable-column">
        {#if leftColumnTiles.length > 0}
            {#each leftColumnTiles as tile, index}
                <div class="tile">
                    <div class="tile-header">{tile.name}, age {tile.age}</div>

                    <div class="tile-budget">Budget: ${tile.budget}</div>
                    <div class="tile-location">Drop-off: {tile.location}</div>
                    <div class="tile-description">{tile.request}</div>
                    <button
                        class="tile-button"
                        on:click={() => selectTile(index)}
                    >
                        Select
                    </button>
                </div>
            {/each}
        {:else}
            <p class="filler-text">
                No gifts to give currently. We saved Christmas!
            </p>
        {/if}
    </div>

    <!-- Main Content Area -->
    <div class="main-content">
        <h3 class="budget">Total Budget: <span id="total-budget">$0</span></h3>

        <div class="bottom-half">
            <div class="delivered-items-section">
                <h3>Delivered Gifts</h3>
                <div class="delivered-items-list">
                    {#if deliveredItems.length === 0}
                        <p class="no-delivered-items">
                            No delivered gifts yet. Help spread the joy!
                        </p>
                    {/if}
                    {#each deliveredItems as item (item.deliveredId)}
                        <div class="delivered-row">
                            <div class="delivered-info">
                                <span>{item.name}</span> –
                                <span>{item.request}</span>
                                <span class="delivered-label">✔ Delivered</span
                                >
                            </div>
                            <button
                                class="undo-button"
                                on:click={() => undoDelivery(item.deliveredId)}
                            >
                                Undo
                            </button>
                        </div>
                    {/each}
                </div>
            </div>
        </div>
    </div>

    <!-- Scrollable Column (Right) -->
    <div class="scrollable-column">
        {#if rightColumnTiles.length > 0}
            {#each rightColumnTiles as tile, index}
                <div class="tile">
                    <div class="tile-header">{tile.name}, age {tile.age}</div>

                    <div class="tile-budget">Budget: ${tile.budget}</div>
                    <div class="tile-location">Drop-off: {tile.location}</div>
                    <div class="tile-description">
                        Asking for: {tile.request}
                    </div>
                    <div class="tile-buttons">
                        <button
                            class="tile-button-delivered"
                            on:click={() => markAsDelivered(index)}
                            >Delivered</button
                        >
                        <button
                            class="tile-button-cancel"
                            on:click={() => cancelTile(index)}>Cancel</button
                        >
                    </div>
                </div>
            {/each}
        {:else}
            <p class="filler-text">
                No gifts selected. Choose a gift and see it here!
            </p>
        {/if}
    </div>
</div>

<style>
    @import url("https://fonts.googleapis.com/css2?family=Snowburst+One&display=swap");
    @import url("https://fonts.googleapis.com/css2?family=Snowburst+One&display=swap");
    @import url("https://fonts.googleapis.com/css2?family=Merienda:wght@300..900&display=swap");

    :global(body) {
        background-color: #f2e8cf;
    }

    .top-bar-labels {
        display: flex;
        justify-content: space-between;
        align-items: center;
        width: 100%;
        position: relative;
    }

    .label {
        font-family: "Merienda", cursive;
        color: white;
        font-size: 1.5rem;
        flex: 1;
        text-align: center;
    }

    .left-label {
        text-align: left;
    }

    .right-label {
        text-align: right;
    }

    .filler-text {
        text-align: center;
        font-family: "Merienda", cursive;
        color: #666;
        font-style: italic;
        margin-top: 24rem;
        font-size: 1rem;
    }

    .top-bar {
        background-color: #386641;
        padding: 1rem;
        display: flex;
        align-items: center;
        justify-content: center;
        color: white;
        border-radius: 15px 15px 0 0;
        position: relative;
        height: 4rem;
    }

    .gift-counter {
        background-color: #bc4749;
        padding: 0.7rem 1.5rem;
        border-radius: 5px;
        color: white;
        font-family: "Merienda", cursive;
        font-size: 2rem;
        font-weight: bold;
        height: 50px;
        width: 600px;
        text-align: center;
    }

    .gift-counter-container {
        text-align: center;
    }

    .layout {
        display: flex;
        height: calc(100vh - 4rem);
    }

    .scrollable-column {
        width: 25%;
        background-color: #a7c957;
        overflow-y: auto;
        padding: 1rem;
        border-left: 3px solid #386641;
        border-right: 3px solid #386641;
    }

    .tile {
        background-color: #6a994e;
        margin-bottom: 1rem;
        border-radius: 15px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        padding: 1rem;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
    }

    .tile-header {
        font-size: 1.2rem;
        font-weight: bold;
        margin-bottom: 0.5rem;
        font-family: "Merienda", cursive;
        color: white;
    }

    .tile-budget {
        font-size: 0.9rem;
        font-weight: bold;
        margin-bottom: 0.5rem;
        font-family: "Merienda", cursive;
        color: white;
    }

    .tile-description {
        font-size: 0.9rem;
        margin-bottom: 0.5rem;
        font-family: "Merienda", cursive;
        color: white;
    }

    .tile-location {
        font-size: 0.9rem;
        margin-bottom: 0.5rem;
        font-family: "Merienda", cursive;
        color: white;
    }

    .tile-buttons {
        display: flex;
        justify-content: flex-end;
        gap: 0.5rem;
        margin-top: auto;
    }

    .tile-button,
    .tile-button-delivered,
    .tile-button-cancel {
        background-color: darkgreen;
        color: white;
        align-self: flex-end;
        margin-top: auto;
        padding: 0.5rem 1rem;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        font-size: 0.9rem;
    }

    .tile-button:hover {
        background-color: #386641;
    }

    .tile-button-delivered {
        background-color: darkgreen;
        color: white;
    }

    .tile-button-delivered:hover {
        background-color: #386641;
    }

    .tile-button-cancel {
        background-color: darkred;
        color: white;
        margin-left: 0.5rem;
    }

    .tile-button-cancel:hover {
        background-color: #bc4749;
    }

    .main-content {
        flex-grow: 1;
        display: flex;
        flex-direction: column;
        padding: 2rem;
        text-align: center;
        background-color: #bc4749;
        border: 2px solid #bc4749;
    }

    .main-content {
        background-color: #bc4749;
        border: 2px solid #bc4749;
        padding: 2rem;
        overflow: auto;
    }

    .delivered-row {
        display: flex;
        justify-content: space-between;
        padding: 0.5rem;
        border-bottom: 1px solid #c3e6cb;
        border-top: 1px solid #c3e6cb;
    }

    .undo-button {
        background-color: #f5c6cb;
        border: none;
        padding: 0.5rem 1rem;
        color: #721c24;
        font-size: 0.9rem;
        border-radius: 5px;
        cursor: pointer;
        margin-right: 15px;
    }

    .undo-button:hover {
        background-color: #f1b0b7;
    }

    .bottom-half {
        flex: 1;
        overflow-y: auto;
        font-family: "Merienda", cursive;
        color: white;
    }

    .delivered-items-list {
        max-height: 600px;
        overflow-y: auto;
        margin-top: 2rem;
    }

    .bottom-half {
        background-color: #6a994e;
        border-radius: 15px;
        margin: 1rem;
    }

    .delivered-info {
        margin-left: 15px;
    }

    .budget {
        font-family: "Merienda", cursive;
        color: white;
        font-size: 2rem;
    }

    .no-delivered-items {
        text-align: center;
        color: #666;
        font-style: italic;
        margin-top: 14rem;
        font-size: 1rem;
    }

    h3 {
        font-size: 1.5rem;
    }
</style>
