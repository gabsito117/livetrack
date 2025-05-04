<!-- === public/index.html === -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Live Checkbox Tracker</title>
  <script src="/socket.io/socket.io.js"></script>
  <style>
    body { font-family: Arial; padding: 20px; }
    .entry { margin: 10px 0; }
    .room { margin: 20px 0; }
  </style>
</head>
<body>
  <h2>Live Checkbox List</h2>

  <!-- Room Selection -->
  <div>
    <label for="roomSelect">Select a Room:</label>
    <select id="roomSelect">
      <option value="Room 1">Room 1</option>
      <option value="Room 2">Room 2</option>
      <!-- More rooms can be added here -->
    </select>
  </div>

  <button id="renameRoomBtn">Rename Room</button>
  <input type="text" id="roomNameInput" placeholder="New room name" style="display:none;" />

  <div id="checkboxList"></div>

  <script>
    const socket = io();
    const roomSelect = document.getElementById("roomSelect");
    const renameRoomBtn = document.getElementById("renameRoomBtn");
    const roomNameInput = document.getElementById("roomNameInput");
    const checkboxList = document.getElementById("checkboxList");

    let currentRoom = roomSelect.value;

    // When room is changed
    roomSelect.addEventListener("change", () => {
      currentRoom = roomSelect.value;
      joinRoom(currentRoom);
    });

    // Join a room
    function joinRoom(roomName) {
      socket.emit("joinRoom", roomName);
    }

    // Rename room
    renameRoomBtn.addEventListener("click", () => {
      roomNameInput.style.display = "block";
      roomNameInput.focus();
    });

    roomNameInput.addEventListener("blur", () => {
      const newRoomName = roomNameInput.value;
      if (newRoomName) {
        socket.emit("renameRoom", { oldName: currentRoom, newName: newRoomName });
        currentRoom = newRoomName;
        roomSelect.innerHTML += `<option value="${newRoomName}">${newRoomName}</option>`;
        roomSelect.value = newRoomName;
      }
      roomNameInput.style.display = "none";
    });

    // Listen for room data
    socket.on("initialState", (names) => {
      checkboxList.innerHTML = "";
      names.forEach(name => {
        const div = document.createElement("div");
        div.classList.add("entry");
        const label = document.createElement("label");
        const checkbox = document.createElement("input");
        checkbox.type = "checkbox";
        checkbox.id = name;
        checkbox.addEventListener("change", () => {
          socket.emit("checkboxChange", { roomName: currentRoom, name, checked: checkbox.checked });
        });
        label.appendChild(checkbox);
        label.append(" " + name);
        div.appendChild(label);
        checkboxList.appendChild(div);
      });
    });

    // Listen for checkbox updates
    socket.on("checkboxUpdate", ({ name, checked }) => {
      const box = document.getElementById(name);
      if (box) box.checked = checked;
    });

    // Initial room join
    joinRoom(currentRoom);
  </script>
</body>
</html>
