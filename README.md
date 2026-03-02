[Xml_Content.xml](https://github.com/user-attachments/files/25675403/Xml_Content.xml)[Reservation.php](https://github.com/user-attachments/files/25675392/Reservation.php)[logout.php](https://github.com/user-attachments/files/25675388/logout.php)[hash_password.php](https://github.com/user-attachments/files/25675383/hash_password.php)[db_functions.php](https://github.com/user-attachments/files/25675370/db_functions.php)# My-Projects  - Hotel Reservation System using XML

[admin.php](https://github.com/user-attachments/files/25675309/admin.php)
<?php
session_start();

// Redirect if already logged in
if (isset($_SESSION['admin_logged_in']) && $_SESSION['admin_logged_in'] === true) {
    // If there's a redirect URL, go there
    if (isset($_SESSION['redirect_url'])) {
        $redirect = $_SESSION['redirect_url'];
        unset($_SESSION['redirect_url']);
        header("Location: $redirect");
        exit();
    }
    header("Location: display.php");
    exit();
}

$error = '';
$success = '';
$showRegister = isset($_GET['register']) || isset($_POST['register_submit']);

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    
    // Handle Registration
    if (isset($_POST['register_submit'])) {
        $username = $_POST['username'] ?? '';
        $password = $_POST['password'] ?? '';
        $confirm_password = $_POST['confirm_password'] ?? '';
        
        // Validation
        if (empty($username) || empty($password) || empty($confirm_password)) {
            $error = 'Please fill in all fields';
        } elseif ($password !== $confirm_password) {
            $error = 'Passwords do not match';
        } elseif (strlen($password) < 6) {
            $error = 'Password must be at least 6 characters long';
        } else {
            // Connect to database
            $conn = new mysqli('localhost', 'root', '', 'hotel_db');
            
            if ($conn->connect_error) {
                die("Connection failed: " . $conn->connect_error);
            }
            
            // Check if username already exists
            $check_stmt = $conn->prepare("SELECT id FROM admin WHERE username = ?");
            $check_stmt->bind_param("s", $username);
            $check_stmt->execute();
            $check_result = $check_stmt->get_result();
            
            if ($check_result->num_rows > 0) {
                $error = 'Username already exists';
            } else {
                // Hash password and insert new admin
                $hashed_password = password_hash($password, PASSWORD_DEFAULT);
                
                $insert_stmt = $conn->prepare("INSERT INTO admin (username, password) VALUES (?, ?)");
                $insert_stmt->bind_param("ss", $username, $hashed_password);
                
                if ($insert_stmt->execute()) {
                    $success = 'Registration successful! You can now login.';
                    $showRegister = false; // Switch back to login form
                } else {
                    $error = 'Registration failed: ' . $conn->error;
                }
                $insert_stmt->close();
            }
            
            $check_stmt->close();
            $conn->close();
        }
    }
    
    // Handle Login
    else {
        $username = $_POST['username'] ?? '';
        $password = $_POST['password'] ?? '';
        
        if (empty($username) || empty($password)) {
            $error = 'Please enter both username and password';
        } else {
            // Connect to database
            $conn = new mysqli('localhost', 'root', '', 'hotel_db');
            
            if ($conn->connect_error) {
                die("Connection failed: " . $conn->connect_error);
            }
            
            // Prepare statement to prevent SQL injection
            $stmt = $conn->prepare("SELECT id, username, password FROM admin WHERE username = ?");
            $stmt->bind_param("s", $username);
            $stmt->execute();
            $result = $stmt->get_result();
            
            if ($result->num_rows === 1) {
                $admin = $result->fetch_assoc();
                
                // Verify password
                if (password_verify($password, $admin['password'])) {
                    $_SESSION['admin_logged_in'] = true;
                    $_SESSION['admin_id'] = $admin['id'];
                    $_SESSION['admin_username'] = $admin['username'];
                    
                    // Redirect to intended page or display
                    if (isset($_GET['redirect'])) {
                        header("Location: " . $_GET['redirect']);
                    } else {
                        header("Location: display.php");
                    }
                    exit();
                } else {
                    $error = 'Invalid username or password';
                }
            } else {
                $error = 'Invalid username or password';
            }
            
            $stmt->close();
            $conn->close();
        }
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin <?php echo $showRegister ? 'Registration' : 'Login'; ?> - Hotel Reservation System</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="login-container">
        <div class="login-header">
            <h1>🔐 Admin <?php echo $showRegister ? 'Registration' : 'Login'; ?></h1>
            <p>Hotel Reservation System</p>
        </div>
        <?php if ($error): ?>
            <div class="error-message"><?php echo htmlspecialchars($error); ?></div>
        <?php endif; ?>
        <?php if ($success): ?>
            <div class="success-message"><?php echo htmlspecialchars($success); ?></div>
        <?php endif; ?>
        <?php if (isset($_GET['expired'])): ?>
            <div class="info-message">Your session has expired. Please login again.</div>
        <?php endif; ?>        
        <?php if ($showRegister): ?>
            <!-- Registration Form -->
            <form method="POST" action="?register=1" class="login-form">
                <div class="form-group">
                    <label for="username">Username</label>
                    <input type="text" id="username" name="username" required 
                           value="<?php echo isset($_POST['username']) ? htmlspecialchars($_POST['username']) : ''; ?>"
                           placeholder="Choose a username">
                </div>  
                <div class="form-group">
                    <label for="password">Password</label>
                    <input type="password" id="password" name="password" required 
                           placeholder="Choose a password (min. 6 characters)">
                    <small style="color: #666; font-size: 12px; display: block; margin-top: 5px;">Minimum 6 characters</small>
                </div>                
                <div class="form-group">
                    <label for="confirm_password">Confirm Password</label>
                    <input type="password" id="confirm_password" name="confirm_password" required 
                           placeholder="Re-enter your password">
                </div>                
                <button type="submit" name="register_submit">Register</button>
                <div class="form-footer">
                    <a href="admin.php" class="btn-link">← Back to Login</a>
                </div>
            </form>
        <?php else: ?>
            <!-- Login Form -->
            <form method="POST" action="" class="login-form">
                <div class="form-group">
                    <label for="username">Username</label>
                    <input type="text" id="username" name="username" required 
                           value="<?php echo isset($_POST['username']) ? htmlspecialchars($_POST['username']) : ''; ?>"
                           placeholder="Enter your username">
                </div>                
                <div class="form-group">
                    <label for="password">Password</label>
                    <input type="password" id="password" name="password" required 
                           placeholder="Enter your password">
                </div>              
                <button type="submit">Login</button>         
                <div class="toggle-form">
                    <a href="?register=1">Don't have an account? Register here</a>
                </div>
            </form>
        <?php endif; ?>    
        <div class="back-link">
            <a href="display.php">← Back to Display View</a>
        </div>
    </div>
</body>
</html>


[auth_check.php](https://github.com/user-attachments/files/25675364/auth_check.php)<?php
session_start();

function checkAdminAuth() {
    // Check if user is logged in
    if (!isset($_SESSION['admin_logged_in']) || $_SESSION['admin_logged_in'] !== true) {
        // Store the requested URL to redirect back after login
        $current_url = (isset($_SERVER['HTTPS']) ? "https" : "http") . "://$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]";
        $_SESSION['redirect_url'] = $current_url;
        
        // Redirect to login page
        header("Location: admin.php?redirect=" . urlencode($current_url));
        exit();
    }
}

// Optional: Check if user has specific role/permissions
function isAdmin() {
    return isset($_SESSION['admin_logged_in']) && $_SESSION['admin_logged_in'] === true;
}

function getCurrentAdmin() {
    if (isAdmin()) {
        return [
            'id' => $_SESSION['admin_id'] ?? null,
            'username' => $_SESSION['admin_username'] ?? null
        ];
    }
    return null;
}
?>






[Uploading db_functions<?php
function getDatabaseConnection() {
    $conn = new mysqli('localhost', 'root', '', 'hotel_db');
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }
    return $conn;
}

function processXmlFile($xmlContent) {
    require_once 'Reservation.php';
    return Reservation($xmlContent);
}

function insertOrUpdateFromXml($xmlContent) {
    $conn = getDatabaseConnection();
    $data = processXmlFile($xmlContent);
    
    $echoToken = $data['OTA_HotelResNotifRQ']['EchoToken'];
    
    // Check if record exists
    $check = $conn->query("SELECT id FROM reservations WHERE echo_token = '$echoToken'");
    $exists = $check->num_rows > 0;
    
    if (!$exists) {
        // Insert new records
        $createDateTime = date('Y-m-d H:i:s', strtotime($data['HotelReservation']['CreateDateTime']));
        
        // Insert into reservations and get the new ID
        $stmt = $conn->prepare("INSERT INTO reservations (echo_token, xmlns, res_status, create_date_time) VALUES (?, ?, ?, ?)");
        $stmt->bind_param("ssss", $echoToken, $data['OTA_HotelResNotifRQ']['xmlns'], $data['HotelReservation']['ResStatus'], $createDateTime);
        $stmt->execute();
        
        $reservationId = $conn->insert_id;
        
        // Insert into rooms using the reservation ID
        $roomStay = $data['HotelReservation']['RoomStays']['RoomStay'];
        $stmt = $conn->prepare("INSERT INTO rooms (id, index_number, hotel_code) VALUES (?, ?, ?)");
        $stmt->bind_param("iis", $reservationId, $roomStay['IndexNumber'], $roomStay['HotelCode']);
        $stmt->execute();
        
        // Insert into guests using the reservation ID
        $resGuest = $data['HotelReservation']['ResGuests']['ResGuest'];
        $stmt = $conn->prepare("INSERT INTO guests (id, res_guest_rph, given_name, surname, telephone, email) VALUES (?, ?, ?, ?, ?, ?)");
        $stmt->bind_param("isssss", $reservationId, $resGuest['ResGuestRPH'], $resGuest['GivenName'], $resGuest['Surname'], $resGuest['PhoneNumber'], $resGuest['Email']);
        $stmt->execute();
        
        // Insert into global_totals using the reservation ID
        $globalInfo = $data['HotelReservation']['ResGlobalInfo'];
        $stmt = $conn->prepare("INSERT INTO global_totals (id, amount_before_tax, amount_after_tax, currency_code) VALUES (?, ?, ?, ?)");
        $stmt->bind_param("idds", $reservationId, $globalInfo['Total_AmountBeforeTax'], $globalInfo['Total_AmountAfterTax'], $globalInfo['Total_CurrencyCode']);
        $stmt->execute();
        
        $conn->close();
        return ["success" => true, "message" => "New entry created successfully! Reservation ID: " . $reservationId, "id" => $reservationId];
        
    } else {
        $reservationData = $check->fetch_assoc();
        $reservationId = $reservationData['id'];
        $needsUpdate = false;
        $updates = [];
        
        // Check reservations table
        $existing = $conn->query("SELECT * FROM reservations WHERE id = $reservationId")->fetch_assoc();
        $currentResStatus = $data['HotelReservation']['ResStatus'];
        $currentXmlns = $data['OTA_HotelResNotifRQ']['xmlns'];
        $currentDateTime = date('Y-m-d H:i:s', strtotime($data['HotelReservation']['CreateDateTime']));
        
        if ($existing['res_status'] != $currentResStatus || $existing['xmlns'] != $currentXmlns || $existing['create_date_time'] != $currentDateTime) {
            $needsUpdate = true;
            $updates[] = "reservations";
            $conn->query("UPDATE reservations SET xmlns='$currentXmlns', res_status='$currentResStatus', create_date_time='$currentDateTime' WHERE id=$reservationId");
        }
        
        // Check rooms table
        $roomResult = $conn->query("SELECT * FROM rooms WHERE id = $reservationId");
        if ($roomResult->num_rows > 0) {
            $existingRoom = $roomResult->fetch_assoc();
            $roomStay = $data['HotelReservation']['RoomStays']['RoomStay'];
            $currentIndex = $roomStay['IndexNumber'];
            $currentHotelCode = $roomStay['HotelCode'];
            
            if ($existingRoom['index_number'] != $currentIndex || $existingRoom['hotel_code'] != $currentHotelCode) {
                $needsUpdate = true;
                $updates[] = "rooms";
                $conn->query("UPDATE rooms SET index_number='$currentIndex', hotel_code='$currentHotelCode' WHERE id=$reservationId");
            }
        }
        
        // Check guests table
        $guestResult = $conn->query("SELECT * FROM guests WHERE id = $reservationId");
        if ($guestResult->num_rows > 0) {
            $existingGuest = $guestResult->fetch_assoc();
            $resGuest = $data['HotelReservation']['ResGuests']['ResGuest'];
            $currentRPH = $resGuest['ResGuestRPH'];
            $currentGivenName = $resGuest['GivenName'];
            $currentSurname = $resGuest['Surname'];
            $currentPhone = $resGuest['PhoneNumber'];
            $currentEmail = $resGuest['Email'];
            
            if ($existingGuest['res_guest_rph'] != $currentRPH || $existingGuest['given_name'] != $currentGivenName || 
                $existingGuest['surname'] != $currentSurname || $existingGuest['telephone'] != $currentPhone || 
                $existingGuest['email'] != $currentEmail) {
                $needsUpdate = true;
                $updates[] = "guests";
                $conn->query("UPDATE guests SET res_guest_rph='$currentRPH', given_name='$currentGivenName', surname='$currentSurname', telephone='$currentPhone', email='$currentEmail' WHERE id=$reservationId");
            }
        }
        
        // Check global_totals table
        $totalResult = $conn->query("SELECT * FROM global_totals WHERE id = $reservationId");
        if ($totalResult->num_rows > 0) {
            $existingTotal = $totalResult->fetch_assoc();
            $globalInfo = $data['HotelReservation']['ResGlobalInfo'];
            $currentBeforeTax = $globalInfo['Total_AmountBeforeTax'];
            $currentAfterTax = $globalInfo['Total_AmountAfterTax'];
            $currentCurrency = $globalInfo['Total_CurrencyCode'];
            
            if ($existingTotal['amount_before_tax'] != $currentBeforeTax || $existingTotal['amount_after_tax'] != $currentAfterTax || $existingTotal['currency_code'] != $currentCurrency) {
                $needsUpdate = true;
                $updates[] = "global_totals";
                $conn->query("UPDATE global_totals SET amount_before_tax='$currentBeforeTax', amount_after_tax='$currentAfterTax', currency_code='$currentCurrency' WHERE id=$reservationId");
            }
        }
        
        $conn->close();
        
        if ($needsUpdate) {
            return ["success" => true, "message" => "Changes detected in: " . implode(", ", $updates) . ". Data updated successfully for Reservation ID: " . $reservationId, "id" => $reservationId];
        } else {
            return ["success" => true, "message" => "No changes detected. Data is already up to date for Reservation ID: " . $reservationId, "id" => $reservationId];
        }
    }
}
?>.php…]()





[display.php](https://github.com/user-attachments/files/25675373/display.php)
<?php
require_once 'auth_check.php';

$conn = new mysqli('localhost', 'root', '', 'hotel_db');

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Get all reservation IDs for dropdown
$reservations = $conn->query("SELECT id, echo_token FROM reservations ORDER BY id DESC");

// Get selected ID from dropdown
$selected_id = isset($_GET['id']) && $_GET['id'] !== '' ? intval($_GET['id']) : null;

// Fetch selected reservation data only if an ID is selected
$reservation_data = null;
if ($selected_id !== null && $selected_id > 0) {
    $reservation = $conn->query("SELECT * FROM reservations WHERE id = $selected_id")->fetch_assoc();
    
    if ($reservation) {
        $room = $conn->query("SELECT * FROM rooms WHERE id = $selected_id")->fetch_assoc();
        $guest = $conn->query("SELECT * FROM guests WHERE id = $selected_id")->fetch_assoc();
        $total = $conn->query("SELECT * FROM global_totals WHERE id = $selected_id")->fetch_assoc();
        
        $reservation_data = [
            'reservation' => $reservation,
            'room' => $room,
            'guest' => $guest,
            'total' => $total
        ];
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hotel Reservation Display</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🏨 Hotel Reservation</h1>
            <div style="display: flex; gap: 10px; align-items: center;">
                <?php if (isAdmin()): ?>
                    <span style="color: #4CAF50; font-size: 14px;">👤 <?php echo htmlspecialchars($_SESSION['admin_username']); ?></span>
                    <a href="logout.php" style="color: #666; text-decoration: none; font-size: 14px;">Logout</a>
                <?php else: ?>
                    <a href="admin.php" style="color: #666; text-decoration: none; font-size: 14px;">Admin Login</a>
                <?php endif; ?>
                
                <?php if ($selected_id && $reservation_data && isAdmin()): ?>
                    <a href="edit.php?id=<?php echo $selected_id; ?>" class="edit-btn">✏️ Edit Reservation</a>
                <?php endif; ?>
            </div>
        </div>
        
        <div class="selector">
            <form method="GET" action="">
                <label for="reservation_id">Select Reservation Invoice:</label>
                <select name="id" id="reservation_id" onchange="this.form.submit()">
                    <option value="">-- Select an invoice --</option>
                    <?php 
                    if ($reservations->num_rows > 0) {
                        while($row = $reservations->fetch_assoc()): 
                    ?>
                        <option value="<?php echo $row['id']; ?>" <?php echo ($selected_id == $row['id']) ? 'selected' : ''; ?>>
                            ID: <?php echo $row['id']; ?> | Token: <?php echo htmlspecialchars($row['echo_token']); ?>
                        </option>
                    <?php 
                        endwhile;
                    } else {
                    ?>
                        <option value="" disabled>No reservations found</option>
                    <?php } ?>
                </select>
                <noscript><button type="submit">View Invoice</button></noscript>
            </form>
        </div>
        
        <?php if ($selected_id === null): ?>
            <div class="invoice">
                <div class="welcome-message">
                    <h3>👋 Welcome to Hotel Reservation System</h3>
                    <p>Please select an invoice from the dropdown menu above to view the details.</p>
                    <?php if (isAdmin()): ?>
                        <p style="margin-top: 20px;">
                            <a href="edit.php" class="create-new-btn">➕ Create New Reservation</a>
                        </p>
                    <?php else: ?>
                        <p style="margin-top: 20px; color: #999; font-size: 14px;">
                            <a href="admin.php" style="color: #4CAF50;">Login as admin</a> to create or edit reservations
                        </p>
                    <?php endif; ?>
                </div>
            </div>
        <?php elseif ($reservation_data): ?>
            <div class="invoice">
                <div class="invoice-header">
                    <h2>RESERVATION INVOICE #<?php echo $selected_id; ?></h2>
                    <div class="id">Token: <?php echo htmlspecialchars($reservation_data['reservation']['echo_token']); ?></div>
                </div>
                
                <div class="section">
                    <h3>📋 Reservation Details</h3>
                    <div class="info-grid">
                        <div class="info-item">
                            <span class="label">Echo Token</span>
                            <span class="value"><?php echo htmlspecialchars($reservation_data['reservation']['echo_token']); ?></span>
                        </div>
                        <div class="info-item">
                            <span class="label">Status</span>
                            <span class="value">
                                <span class="status-badge status-<?php echo strtolower($reservation_data['reservation']['res_status']); ?>">
                                    <?php echo htmlspecialchars($reservation_data['reservation']['res_status']); ?>
                                </span>
                            </span>
                        </div>
                        <div class="info-item">
                            <span class="label">Website</span>
                            <span class="value"><?php echo htmlspecialchars($reservation_data['reservation']['xmlns']); ?></span>
                        </div>
                        
                        <div class="info-item">
                            <span class="label">Created At</span>
                            <span class="value"><?php echo date('F j, Y H:i:s', strtotime($reservation_data['reservation']['created_at'])); ?></span>
                        </div>
                    </div>
                </div>
                
                <div class="section">
                    <h3>🏠 Room Information</h3>
                    <div class="info-grid">
                        <div class="info-item">
                            <span class="label">Room Index</span>
                            <span class="value"><?php echo htmlspecialchars($reservation_data['room']['index_number']); ?></span>
                        </div>
                        <div class="info-item">
                            <span class="label">Hotel Code</span>
                            <span class="value"><?php echo htmlspecialchars($reservation_data['room']['hotel_code']); ?></span>
                        </div>
                    </div>
                </div>
                
                <div class="section">
                    <h3>👤 Guest Information</h3>
                    <div class="info-grid">
                        <div class="info-item">
                            <span class="label">Guest RPH</span>
                            <span class="value"><?php echo htmlspecialchars($reservation_data['guest']['res_guest_rph']); ?></span>
                        </div>
                        <div class="info-item">
                            <span class="label">Full Name</span>
                            <span class="value"><?php echo htmlspecialchars($reservation_data['guest']['given_name'] . ' ' . $reservation_data['guest']['surname']); ?></span>
                        </div>
                        <div class="info-item">
                            <span class="label">Telephone</span>
                            <span class="value"><?php echo !empty($reservation_data['guest']['telephone']) ? htmlspecialchars($reservation_data['guest']['telephone']) : 'Not provided'; ?></span>
                        </div>
                        <div class="info-item">
                            <span class="label">Email</span>
                            <span class="value"><?php echo !empty($reservation_data['guest']['email']) ? htmlspecialchars($reservation_data['guest']['email']) : 'Not provided'; ?></span>
                        </div>
                    </div>
                </div>
                
                <div class="section">
                    <h3>💰 Payment Summary</h3>
                    <div class="total-box">
                        <div class="total-row">
                            <span>Amount Before Tax:</span>
                            <span><?php echo number_format($reservation_data['total']['amount_before_tax'], 2); ?> <?php echo htmlspecialchars($reservation_data['total']['currency_code']); ?></span>
                        </div>
                        <div class="total-row">
                            <span>Amount After Tax:</span>
                            <span><?php echo number_format($reservation_data['total']['amount_after_tax'], 2); ?> <?php echo htmlspecialchars($reservation_data['total']['currency_code']); ?></span>
                        </div>
                        <div class="total-row">
                            <span>Tax Amount:</span>
                            <span><?php echo number_format($reservation_data['total']['amount_after_tax'] - $reservation_data['total']['amount_before_tax'], 2); ?> <?php echo htmlspecialchars($reservation_data['total']['currency_code']); ?></span>
                        </div>
                        <div class="total-row">
                            <span>TOTAL AMOUNT:</span>
                            <span><?php echo number_format($reservation_data['total']['amount_after_tax'], 2); ?> <?php echo htmlspecialchars($reservation_data['total']['currency_code']); ?></span>
                        </div>
                    </div>
                </div>
            </div>
        <?php else: ?>
            <!-- Invalid or non-existent invoice ID -->
            <div class="invoice">
                <div class="no-data">
                    <h3>❌ Invoice Not Found</h3>
                    <p>No reservation found with ID: <?php echo $selected_id; ?></p>
                    <p style="margin-top: 10px; font-size: 14px;">Please select a valid invoice from the dropdown menu.</p>
                </div>
            </div>
        <?php endif; ?>
    </div>
</body>
</html>
<?php $conn->close(); ?>






[edit.php](https://github.com/user-attachments/files/25675382/edit.php)<?php
require_once 'auth_check.php';
checkAdminAuth(); // This will redirect to login if not authenticated

$conn = new mysqli('localhost', 'root', '', 'hotel_db');

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$message = '';
$messageType = '';
$reservation_id = isset($_GET['id']) ? intval($_GET['id']) : null;

// Handle delete action
if (isset($_POST['delete']) && $_POST['delete'] == 'yes' && $reservation_id) {
    // First check if reservation exists
    $check = $conn->query("SELECT id FROM reservations WHERE id = $reservation_id");
    if ($check->num_rows > 0) {
        // Delete will cascade to other tables due to FOREIGN KEY constraints
        $stmt = $conn->prepare("DELETE FROM reservations WHERE id = ?");
        $stmt->bind_param("i", $reservation_id);
        if ($stmt->execute()) {
            $message = "Reservation #$reservation_id deleted successfully!";
            $messageType = 'success';
            $reservation_id = null; // Clear the ID after deletion
        } else {
            $message = "Error deleting reservation: " . $conn->error;
            $messageType = 'error';
        }
    }
}

// Handle form submission
if ($_SERVER['REQUEST_METHOD'] === 'POST' && !isset($_POST['delete'])) {
    $echoToken = $_POST['echo_token'];
    $xmlns = $_POST['xmlns'];
    $resStatus = $_POST['res_status'];
    $createDateTime = $_POST['create_date_time'];
    
    // Room data
    $indexNumber = $_POST['index_number'];
    $hotelCode = $_POST['hotel_code'];
    
    // Guest data
    $resGuestRPH = $_POST['res_guest_rph'];
    $givenName = $_POST['given_name'];
    $surname = $_POST['surname'];
    $telephone = $_POST['telephone'];
    $email = $_POST['email'];
    
    // Total data
    $amountBeforeTax = $_POST['amount_before_tax'];
    $amountAfterTax = $_POST['amount_after_tax'];
    $currencyCode = $_POST['currency_code'];
    
    // Format datetime for MySQL
    $mysqlDateTime = date('Y-m-d H:i:s', strtotime($createDateTime));
    
    // Check if token exists (excluding current reservation if editing)
    $tokenCheck = "SELECT id FROM reservations WHERE echo_token = '$echoToken'";
    if ($reservation_id) {
        $tokenCheck .= " AND id != $reservation_id";
    }
    $check = $conn->query($tokenCheck);
    $tokenExists = $check->num_rows > 0;
    
    if ($tokenExists) {
        $message = "Error: Echo Token '$echoToken' already exists with a different reservation ID!";
        $messageType = 'error';
    } else {
        if ($reservation_id) {
            // UPDATE existing reservation
            $conn->begin_transaction();
            
            try {
                // Update reservations table
                $stmt = $conn->prepare("UPDATE reservations SET echo_token=?, xmlns=?, res_status=?, create_date_time=? WHERE id=?");
                $stmt->bind_param("ssssi", $echoToken, $xmlns, $resStatus, $mysqlDateTime, $reservation_id);
                $stmt->execute();
                
                // Update rooms table
                $stmt = $conn->prepare("UPDATE rooms SET index_number=?, hotel_code=? WHERE id=?");
                $stmt->bind_param("isi", $indexNumber, $hotelCode, $reservation_id);
                $stmt->execute();
                
                // Update guests table
                $stmt = $conn->prepare("UPDATE guests SET res_guest_rph=?, given_name=?, surname=?, telephone=?, email=? WHERE id=?");
                $stmt->bind_param("sssssi", $resGuestRPH, $givenName, $surname, $telephone, $email, $reservation_id);
                $stmt->execute();
                
                // Update global_totals table
                $stmt = $conn->prepare("UPDATE global_totals SET amount_before_tax=?, amount_after_tax=?, currency_code=? WHERE id=?");
                $stmt->bind_param("ddsi", $amountBeforeTax, $amountAfterTax, $currencyCode, $reservation_id);
                $stmt->execute();
                
                $conn->commit();
                $message = "Reservation #$reservation_id updated successfully!";
                $messageType = 'success';
                
                // Refresh the page to show updated data
                echo "<script>window.location.href = 'edit.php?id=$reservation_id&updated=1';</script>";
                exit();
                
            } catch (Exception $e) {
                $conn->rollback();
                $message = "Error updating reservation: " . $e->getMessage();
                $messageType = 'error';
            }
        } else {
            // CREATE new reservation
            $conn->begin_transaction();
            
            try {
                // Insert into reservations
                $stmt = $conn->prepare("INSERT INTO reservations (echo_token, xmlns, res_status, create_date_time) VALUES (?, ?, ?, ?)");
                $stmt->bind_param("ssss", $echoToken, $xmlns, $resStatus, $mysqlDateTime);
                $stmt->execute();
                
                $newId = $conn->insert_id;
                
                // Insert into rooms
                $stmt = $conn->prepare("INSERT INTO rooms (id, index_number, hotel_code) VALUES (?, ?, ?)");
                $stmt->bind_param("iis", $newId, $indexNumber, $hotelCode);
                $stmt->execute();
                
                // Insert into guests
                $stmt = $conn->prepare("INSERT INTO guests (id, res_guest_rph, given_name, surname, telephone, email) VALUES (?, ?, ?, ?, ?, ?)");
                $stmt->bind_param("isssss", $newId, $resGuestRPH, $givenName, $surname, $telephone, $email);
                $stmt->execute();
                
                // Insert into global_totals
                $stmt = $conn->prepare("INSERT INTO global_totals (id, amount_before_tax, amount_after_tax, currency_code) VALUES (?, ?, ?, ?)");
                $stmt->bind_param("idds", $newId, $amountBeforeTax, $amountAfterTax, $currencyCode);
                $stmt->execute();
                
                $conn->commit();
                $message = "New reservation created successfully! ID: " . $newId;
                $messageType = 'success';
                
                // Redirect to edit page for the new reservation
                echo "<script>window.location.href = 'edit.php?id=$newId&created=1';</script>";
                exit();
                
            } catch (Exception $e) {
                $conn->rollback();
                $message = "Error creating reservation: " . $e->getMessage();
                $messageType = 'error';
            }
        }
    }
}

// Check for success messages from redirects
if (isset($_GET['updated'])) {
    $message = "Reservation #$reservation_id updated successfully!";
    $messageType = 'success';
} elseif (isset($_GET['created'])) {
    $message = "New reservation created successfully! ID: $reservation_id";
    $messageType = 'success';
}

// Fetch existing data if editing
$editData = null;
if ($reservation_id) {
    $reservation = $conn->query("SELECT * FROM reservations WHERE id = $reservation_id")->fetch_assoc();
    if ($reservation) {
        $room = $conn->query("SELECT * FROM rooms WHERE id = $reservation_id")->fetch_assoc();
        $guest = $conn->query("SELECT * FROM guests WHERE id = $reservation_id")->fetch_assoc();
        $total = $conn->query("SELECT * FROM global_totals WHERE id = $reservation_id")->fetch_assoc();
        
        $editData = [
            'reservation' => $reservation,
            'room' => $room,
            'guest' => $guest,
            'total' => $total
        ];
    }
}

// Get all reservations for dropdown
$reservations = $conn->query("SELECT id, echo_token FROM reservations ORDER BY id DESC");
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?php echo $editData ? 'Edit' : 'Create'; ?> Reservation</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <!-- Delete Confirmation Modal -->
    <div id="deleteModal" class="modal">
        <div class="modal-content">
            <h3>⚠️ Confirm Delete</h3>
            <p>Are you sure you want to delete this reservation? This action cannot be undone.</p>
            <div class="modal-actions">
                <form method="POST" action="edit.php?id=<?php echo $reservation_id; ?>" style="display: inline;">
                    <input type="hidden" name="delete" value="yes">
                    <button type="submit" class="btn btn-danger">Yes, Delete</button>
                </form>
                <button onclick="document.getElementById('deleteModal').style.display='none'" class="btn btn-secondary">Cancel</button>
            </div>
        </div>
    </div>

    <div class="container">
        <div class="header">
            <h1><?php echo $editData ? '✏️ Edit Reservation #' . $reservation_id : 'Create New Reservation'; ?></h1>
            <div class="button-group">
                <?php if ($editData): ?>
                    <button onclick="document.getElementById('deleteModal').style.display='block'" class="delete-btn">🗑️ Delete</button>
                <?php endif; ?>
                <a href="display.php" class="edit-btn">← Back to Display</a>
            </div>
        </div>
        
        <!-- Quick reservation switcher -->
        <?php if ($reservations && $reservations->num_rows > 0): ?>
        <div class="selector reservation-selector">
            <form method="GET" action="edit.php">
                <label for="reservation_id">Switch to another reservation:</label>
                <select name="id" id="reservation_id" onchange="this.form.submit()">
                    <option value="">-- Create New --</option>
                    <?php 
                    $reservations->data_seek(0);
                    while($row = $reservations->fetch_assoc()): 
                    ?>
                        <option value="<?php echo $row['id']; ?>" <?php echo ($reservation_id == $row['id']) ? 'selected' : ''; ?>>
                            ID: <?php echo $row['id']; ?> | Token: <?php echo htmlspecialchars($row['echo_token']); ?>
                        </option>
                    <?php endwhile; ?>
                </select>
                <noscript><button type="submit">Switch</button></noscript>
            </form>
        </div>
        <?php endif; ?>
        
        <?php if ($message): ?>
            <div class="message <?php echo $messageType; ?>">
                <?php echo $message; ?>
            </div>
        <?php endif; ?>
        
        <div class="edit-form">
            <?php if ($reservation_id && !$editData && !$message): ?>
                <div class="message error">
                    Reservation not found with ID: <?php echo $reservation_id; ?>
                </div>
            <?php endif; ?>
            
            <form method="POST" action="edit.php<?php echo $reservation_id ? '?id=' . $reservation_id : ''; ?>">
                <?php if ($editData): ?>
                    <input type="hidden" name="reservation_id" value="<?php echo $reservation_id; ?>">
                <?php endif; ?>
                
                <?php if ($editData): ?>
                <div class="token-warning">
                    ⚠️ Warning: Changing the Echo Token will update this reservation. If you want to create a new reservation with this data, please go back and click "Create New".
                </div>
                <?php endif; ?>
                
                <!-- Reservation Details -->
                <div class="form-section">
                    <h3>📋 Reservation Details</h3>
                    <div class="form-row">
                        <div class="form-group">
                            <label>Echo Token *</label>
                            <input type="number" name="echo_token" required 
                                   value="<?php echo $editData ? htmlspecialchars($editData['reservation']['echo_token']) : ''; ?>">
                        </div>
                        <div class="form-group">
                            <label>Status *</label>
                            <input type="text" name="res_status" required 
                                   value="<?php echo $editData ? htmlspecialchars($editData['reservation']['res_status']) : 'Commit'; ?>">
                        </div>
                    </div>
                    <div class="form-row">
                        <div class="form-group">
                            <label>XML Namespace</label>
                            <input type="text" name="xmlns" 
                                   value="<?php echo $editData ? htmlspecialchars($editData['reservation']['xmlns']) : 'http://www.opentravel.org/OTA/2003/05'; ?>">
                        </div>
                        <div class="form-group">
                            <label>Create Date Time *</label>
                            <input type="datetime-local" name="create_date_time" required 
                                   value="<?php echo $editData ? date('Y-m-d\TH:i', strtotime($editData['reservation']['create_date_time'])) : ''; ?>">
                        </div>
                    </div>
                </div>
                
                <!-- Room Information -->
                <div class="form-section">
                    <h3>🏠 Room Information</h3>
                    <div class="form-row">
                        <div class="form-group">
                            <label>Index Number *</label>
                            <input type="number" name="index_number" required 
                                   value="<?php echo $editData ? htmlspecialchars($editData['room']['index_number']) : '1'; ?>">
                        </div>
                        <div class="form-group">
                            <label>Hotel Code *</label>
                            <input type="text" name="hotel_code" required 
                                   value="<?php echo $editData ? htmlspecialchars($editData['room']['hotel_code']) : ''; ?>">
                        </div>
                    </div>
                </div>
                
                <!-- Guest Information -->
                <div class="form-section">
                    <h3>👤 Guest Information</h3>
                    <div class="form-row">
                        <div class="form-group">
                            <label>Guest RPH *</label>
                            <input type="number" name="res_guest_rph" required 
                                   value="<?php echo $editData ? htmlspecialchars($editData['guest']['res_guest_rph']) : '0'; ?>">
                        </div>
                        <div class="form-group">
                        <label>Given Name *</label>
                        <input type="text" name="given_name" required 
                               value="<?php echo $editData ? htmlspecialchars($editData['guest']['given_name']) : ''; ?>"
                               onkeypress="return (event.charCode > 64 && event.charCode < 91) || (event.charCode > 96 && event.charCode < 123) || event.charCode == 32"
                               pattern="[A-Za-z ]+" 
                               title="Only letters and spaces allowed">
                        </div>
                    </div>
                    <div class="form-row">
                        <div class="form-group">
                        <label>Surname *</label>
                        <input type="text" name="surname" required 
                               value="<?php echo $editData ? htmlspecialchars($editData['guest']['surname']) : ''; ?>"
                               onkeypress="return (event.charCode > 64 && event.charCode < 91) || (event.charCode > 96 && event.charCode < 123) || event.charCode == 32"
                               pattern="[A-Za-z ]+" 
                               title="Only letters and spaces allowed">
                        </div>
                        <div class="form-group">
                            <label>Telephone</label>
                            <input type="text" name="telephone" 
                                   value="<?php echo $editData ? htmlspecialchars($editData['guest']['telephone']) : ''; ?>">
                        </div>
                    </div>
                    <div class="form-group">
                        <label>Email</label>
                        <input type="email" name="email" 
                               value="<?php echo $editData ? htmlspecialchars($editData['guest']['email']) : ''; ?>">
                    </div>
                </div>
                
                <!-- Payment Information -->
                <div class="form-section">
                    <h3>💰 Payment Information</h3>
                    <div class="form-row">
                        <div class="form-group">
                            <label>Amount Before Tax *</label>
                            <input type="number" step="0.01" name="amount_before_tax" required 
                                   value="<?php echo $editData ? htmlspecialchars($editData['total']['amount_before_tax']) : ''; ?>">
                        </div>
                        <div class="form-group">
                            <label>Amount After Tax *</label>
                            <input type="number" step="0.01" name="amount_after_tax" required 
                                   value="<?php echo $editData ? htmlspecialchars($editData['total']['amount_after_tax']) : ''; ?>">
                        </div>
                    </div>
                    <div class="form-group">
                        <label>Currency Code *</label>
                        <input type="text" name="currency_code" required 
                               value="<?php echo $editData ? htmlspecialchars($editData['total']['currency_code']) : 'IDR'; ?>">
                    </div>
                </div>
                
                <div class="form-actions">
                    <button type="submit" class="btn btn-primary">
                        <?php echo $editData ? 'Update Reservation' : 'Create Reservation'; ?>
                    </button>
                    <a href="display.php" class="btn btn-secondary">Cancel</a>
                </div>
            </form>
        </div>
    </div>

    <script>
        // Close modal if user clicks outside of it
        window.onclick = function(event) {
            var modal = document.getElementById('deleteModal');
            if (event.target == modal) {
                modal.style.display = 'none';
            }
        }
    </script>
</body>
</html>
<?php $conn->close(); ?>



[Uploading hash_password.php<?php
echo password_hash('admin123', PASSWORD_DEFAULT);
?>…]()









[insert.php](https://github.com/user-attachments/files/25675386/insert.php)
<?php
require_once 'db_functions.php';

$xmlContent = file_get_contents('Xml_Content.xml');
$result = insertOrUpdateFromXml($xmlContent);

if ($result['success']) {
    echo $result['message'];
} else {
    echo "Error: " . $result['message'];
}

$conn = new mysqli('localhost', 'root', '', 'hotel_db');

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$xmlContent = file_get_contents('Xml_Content.xml');
$data = Reservation($xmlContent);

$echoToken = $data['OTA_HotelResNotifRQ']['EchoToken'];

// Check if record exists
$check = $conn->query("SELECT id FROM reservations WHERE echo_token = '$echoToken'");
$exists = $check->num_rows > 0;

if (!$exists) {
    // Insert new records
    // echo "New EchoToken detected. Creating new entry...<br>";
    
    $createDateTime = date('Y-m-d H:i:s', strtotime($data['HotelReservation']['CreateDateTime']));
    
    // Insert into reservations and get the new ID
    $stmt = $conn->prepare("INSERT INTO reservations (echo_token, xmlns, res_status, create_date_time) VALUES (?, ?, ?, ?)");
    $stmt->bind_param("isss", $echoToken, $data['OTA_HotelResNotifRQ']['xmlns'], $data['HotelReservation']['ResStatus'], $createDateTime);
    $stmt->execute();
    
    $reservationId = $conn->insert_id;
    
    // Insert into rooms using the reservation ID
    $roomStay = $data['HotelReservation']['RoomStays']['RoomStay'];
    $stmt = $conn->prepare("INSERT INTO rooms (id, index_number, hotel_code) VALUES (?, ?, ?)");
    $stmt->bind_param("iis", $reservationId, $roomStay['IndexNumber'], $roomStay['HotelCode']);
    $stmt->execute();
    
    // Insert into guests using the reservation ID
    $resGuest = $data['HotelReservation']['ResGuests']['ResGuest'];
    $stmt = $conn->prepare("INSERT INTO guests (id, res_guest_rph, given_name, surname, telephone, email) VALUES (?, ?, ?, ?, ?, ?)");
    $stmt->bind_param("isssss", $reservationId, $resGuest['ResGuestRPH'], $resGuest['GivenName'], $resGuest['Surname'], $resGuest['PhoneNumber'], $resGuest['Email']);
    $stmt->execute();
    
    // Insert into global_totals using the reservation ID
    $globalInfo = $data['HotelReservation']['ResGlobalInfo'];
    $stmt = $conn->prepare("INSERT INTO global_totals (id, amount_before_tax, amount_after_tax, currency_code) VALUES (?, ?, ?, ?)");
    $stmt->bind_param("idds", $reservationId, $globalInfo['Total_AmountBeforeTax'], $globalInfo['Total_AmountAfterTax'], $globalInfo['Total_CurrencyCode']);
    $stmt->execute();
    
    echo "New entry created successfully! Reservation ID: " . $reservationId;
    
} else {
    $reservationData = $check->fetch_assoc();
    $reservationId = $reservationData['id'];
    $needsUpdate = false;
    $updates = [];
    
    // Check reservations table
    $existing = $conn->query("SELECT * FROM reservations WHERE id = $reservationId")->fetch_assoc();
    $currentResStatus = $data['HotelReservation']['ResStatus'];
    $currentXmlns = $data['OTA_HotelResNotifRQ']['xmlns'];
    $currentDateTime = date('Y-m-d H:i:s', strtotime($data['HotelReservation']['CreateDateTime']));
    
    if ($existing['res_status'] != $currentResStatus || $existing['xmlns'] != $currentXmlns || $existing['create_date_time'] != $currentDateTime) {
        $needsUpdate = true;
        $updates[] = "reservations";
        $conn->query("UPDATE reservations SET xmlns='$currentXmlns', res_status='$currentResStatus', create_date_time='$currentDateTime' WHERE id=$reservationId");
    }
    
    // Check rooms table
    $roomResult = $conn->query("SELECT * FROM rooms WHERE id = $reservationId");
    if ($roomResult->num_rows > 0) {
        $existingRoom = $roomResult->fetch_assoc();
        $roomStay = $data['HotelReservation']['RoomStays']['RoomStay'];
        $currentIndex = $roomStay['IndexNumber'];
        $currentHotelCode = $roomStay['HotelCode'];
        
        if ($existingRoom['index_number'] != $currentIndex || $existingRoom['hotel_code'] != $currentHotelCode) {
            $needsUpdate = true;
            $updates[] = "rooms";
            $conn->query("UPDATE rooms SET index_number='$currentIndex', hotel_code='$currentHotelCode' WHERE id=$reservationId");
        }
    }
    
    // Check guests table
    $guestResult = $conn->query("SELECT * FROM guests WHERE id = $reservationId");
    if ($guestResult->num_rows > 0) {
        $existingGuest = $guestResult->fetch_assoc();
        $resGuest = $data['HotelReservation']['ResGuests']['ResGuest'];
        $currentRPH = $resGuest['ResGuestRPH'];
        $currentGivenName = $resGuest['GivenName'];
        $currentSurname = $resGuest['Surname'];
        $currentPhone = $resGuest['PhoneNumber'];
        $currentEmail = $resGuest['Email'];
        
        if ($existingGuest['res_guest_rph'] != $currentRPH || $existingGuest['given_name'] != $currentGivenName || 
            $existingGuest['surname'] != $currentSurname || $existingGuest['telephone'] != $currentPhone || 
            $existingGuest['email'] != $currentEmail) {
            $needsUpdate = true;
            $updates[] = "guests";
            $conn->query("UPDATE guests SET res_guest_rph='$currentRPH', given_name='$currentGivenName', surname='$currentSurname', telephone='$currentPhone', email='$currentEmail' WHERE id=$reservationId");
        }
    }
    
    // Check global_totals table
    $totalResult = $conn->query("SELECT * FROM global_totals WHERE id = $reservationId");
    if ($totalResult->num_rows > 0) {
        $existingTotal = $totalResult->fetch_assoc();
        $globalInfo = $data['HotelReservation']['ResGlobalInfo'];
        $currentBeforeTax = $globalInfo['Total_AmountBeforeTax'];
        $currentAfterTax = $globalInfo['Total_AmountAfterTax'];
        $currentCurrency = $globalInfo['Total_CurrencyCode'];
        
        if ($existingTotal['amount_before_tax'] != $currentBeforeTax || $existingTotal['amount_after_tax'] != $currentAfterTax || $existingTotal['currency_code'] != $currentCurrency) {
            $needsUpdate = true;
            $updates[] = "global_totals";
            $conn->query("UPDATE global_totals SET amount_before_tax='$currentBeforeTax', amount_after_tax='$currentAfterTax', currency_code='$currentCurrency' WHERE id=$reservationId");
        }
    }
    
    // // Show result
    // if ($needsUpdate) {
    //     echo "Changes detected in: " . implode(", ", $updates) . "<br>";
    //     echo "Data updated successfully for Reservation ID: " . $reservationId;
    // } else {
    //     echo "No changes detected. Data is already up to date for Reservation ID: " . $reservationId;
    // }
}

$conn->close();
?>





[Uploading logout.php…<?php
session_start();
session_destroy();
header("Location: admin.php?logged_out=1");
exit();
?>]()





[Uploading Reservation.<?php
$file = 'Xml_Content.xml';
$xmlContent = file_get_contents($file) or exit("Failed to open $file.");
// if (preg_match('/\.xml$/i', $file)) {
//     echo "File is an XML file.<br>";
// }

function Reservation($xml) {
    $data = [];
    
    // OTA
    preg_match('/<OTA_HotelResNotifRQ\s+EchoToken="([^"]*)"\s+TimeStamp="([^"]*)"\s+Version="([^"]*)"\s+xmlns="([^"]*)"[^>]*>/s', $xml, $matches);
    $data['OTA_HotelResNotifRQ'] = [
        'EchoToken' => $matches[1] ?? '',
        'TimeStamp' => $matches[2] ?? '',
        'xmlns' => $matches[4] ?? ''
    ];
    
    // POS 
    preg_match('/<POS>\s*<Source>\s*<BookingChannel Type="([^"]*)">\s*<CompanyName Code="([^"]*)">([^<]*)<\/CompanyName>\s*<\/BookingChannel>\s*<\/Source>\s*<\/POS>/s', $xml, $matches);
    $data['POS'] = [
        'BookingChannel_Type' => $matches[1] ?? '',
        'CompanyName_Value' => $matches[3] ?? ''
    ];
    
    // HotelReservation
    preg_match('/<HotelReservation\s+ResStatus="([^"]*)"\s+CreateDateTime="([^"]*)">/s', $xml, $matches);
    $data['HotelReservation'] = [
        'ResStatus' => $matches[1] ?? '',
        'CreateDateTime' => $matches[2] ?? ''
    ];
    
    // RoomStays
    $data['HotelReservation']['RoomStays'] = [];
    
    // RoomStay - flattened key-value pairs
    preg_match('/<RoomStay\s+IndexNumber="([^"]*)">/s', $xml, $matches);
    $roomStayData = [
        'IndexNumber' => $matches[1] ?? ''
    ];
    
    // RatePlanDescription
    preg_match('/<RatePlanDescription>\s*<Text>([^<]*)<\/Text>\s*<\/RatePlanDescription>/s', $xml, $matches);
    $roomStayData['RatePlanDescription_Text'] = $matches[1] ?? '';
    
    // Commission
    preg_match('/<CommissionableAmount\s+CurrencyCode="([^"]*)"\s+Amount="([^"]*)"\/>/s', $xml, $matches);
    $roomStayData['Commission_CurrencyCode'] = $matches[1] ?? '';
    $roomStayData['Commission_Amount'] = $matches[2] ?? '';
    
    // GuestCounts
    preg_match('/<GuestCount\s+AgeQualifyingCode="([^"]*)"\s+Count="([^"]*)"\/>/s', $xml, $matches);
    $roomStayData['GuestCount_Count'] = $matches[2] ?? '';
    
    // TimeSpan
    preg_match('/<TimeSpan\s+Start="([^"]*)"\s+End="([^"]*)"\/>/s', $xml, $matches);
    $roomStayData['TimeSpan_Start'] = $matches[1] ?? '';
    $roomStayData['TimeSpan_End'] = $matches[2] ?? '';
    
    // Total (RoomStay)
    preg_match('/<Total\s+AmountBeforeTax="([^"]*)"\s+AmountAfterTax="([^"]*)"\s+CurrencyCode="([^"]*)"\/>/s', $xml, $matches);
    $roomStayData['Total_AmountAfterTax'] = $matches[2] ?? '';
    $roomStayData['Total_CurrencyCode'] = $matches[3] ?? '';
    
    // BasicPropertyInfo
    preg_match('/<BasicPropertyInfo\s+HotelCode="([^"]*)"\/>/s', $xml, $matches);
    $roomStayData['HotelCode'] = $matches[1] ?? '';
    
    $data['HotelReservation']['RoomStays']['RoomStay'] = $roomStayData;
    
    // ResGuests
    $data['HotelReservation']['ResGuests'] = [];
    
    preg_match('/<ResGuest\s+ResGuestRPH="([^"]*)">/s', $xml, $matches);
    $resGuestData = [
        'ResGuestRPH' => $matches[1] ?? ''
    ];
    
    // Profiles (PersonName)
    preg_match('/<GivenName>([^<]*)<\/GivenName>\s*<Surname>([^<]*)<\/Surname>/s', $xml, $matches);
    $resGuestData['GivenName'] = $matches[1] ?? '';
    $resGuestData['Surname'] = $matches[2] ?? '';
    
    // Telephone
    preg_match('/<Telephone\s+PhoneNumber="([^"]*)"\/>/s', $xml, $matches);
    $resGuestData['PhoneNumber'] = $matches[1] ?? '';
    
    // Email
    preg_match('/<Email>([^<]*)<\/Email>/s', $xml, $matches);
    $resGuestData['Email'] = $matches[1] ?? '';
    
    $data['HotelReservation']['ResGuests']['ResGuest'] = $resGuestData;
    
    // ResGlobalInfo - flattened
    $data['HotelReservation']['ResGlobalInfo'] = [];
    
    // Total (ResGlobalInfo)
    preg_match_all('/<Total\s+AmountBeforeTax="([^"]*)"\s+AmountAfterTax="([^"]*)"\s+CurrencyCode="([^"]*)"\/>/s', $xml, $totalMatches, PREG_SET_ORDER);
    if (isset($totalMatches[1])) {
        $data['HotelReservation']['ResGlobalInfo']['Total_AmountBeforeTax'] = $totalMatches[1][1] ?? '';
        $data['HotelReservation']['ResGlobalInfo']['Total_AmountAfterTax'] = $totalMatches[1][2] ?? '';
        $data['HotelReservation']['ResGlobalInfo']['Total_CurrencyCode'] = $totalMatches[1][3] ?? '';
    }
    
    // HotelReservationID
    preg_match('/<HotelReservationID\s+ResID_Type="([^"]*)"\s+ResID_Value="([^"]*)"\/>/s', $xml, $matches);
    $data['HotelReservation']['ResGlobalInfo']['ResID_Value'] = $matches[2] ?? '';
    
    return $data;
}

// echo "<pre>";
// print_r(Reservation($xmlContent));
// echo "</pre>";

?>php…]()








[reservation.sql](https://github.com/user-attachments/files/25675394/reservation.sql)
CREATE TABLE reservations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    echo_token VARCHAR(255) UNIQUE,
    xmlns VARCHAR(255),
    res_status VARCHAR(50),
    create_date_time DATETIME,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE rooms (
    id INT PRIMARY KEY,
    index_number INT,
    hotel_code VARCHAR(100),
    FOREIGN KEY (id) REFERENCES reservations(id) ON DELETE CASCADE
);

CREATE TABLE guests (
    id INT PRIMARY KEY,
    res_guest_rph VARCHAR(50),
    given_name VARCHAR(255),
    surname VARCHAR(255),
    telephone VARCHAR(50),
    email VARCHAR(255),
    FOREIGN KEY (id) REFERENCES reservations(id) ON DELETE CASCADE
);

CREATE TABLE global_totals (
    id INT PRIMARY KEY,
    amount_before_tax DECIMAL(10,2),
    amount_after_tax DECIMAL(10,2),
    currency_code VARCHAR(10),
    FOREIGN KEY (id) REFERENCES reservations(id) ON DELETE CASCADE
);

CREATE TABLE admin (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert a default admin user (password: admin123)
INSERT INTO admin (username, password) 
VALUES ('admin', '$2y$10$C/26pdZneCQKPxoEQszx1eA7qf3RIhA4bb3iANj5oELavOQzVdCN6');






[style.css](https://github.com/user-attachments/files/25675399/style.css)* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    background: #f5f5f5;
    padding: 20px;
}

.container {
    max-width: 800px;
    margin: 0 auto;
}

.header {
    background: white;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.header h1 {
    color: #333;
    font-size: 24px;
}

.refresh-info {
    color: #666;
    font-size: 14px;
}

.selector {
    background: white;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.selector label {
    display: block;
    margin-bottom: 8px;
    color: #555;
    font-weight: bold;
}

.selector select {
    width: 100%;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 16px;
    margin-bottom: 10px;
}

.selector button {
    background: #4CAF50;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
}

.selector button:hover {
    background: #45a049;
}

.invoice {
    background: white;
    border-radius: 8px;
    padding: 30px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.invoice-header {
    text-align: center;
    margin-bottom: 30px;
    padding-bottom: 20px;
    border-bottom: 2px solid #eee;
}

.invoice-header h2 {
    color: #333;
    margin-bottom: 10px;
}

.invoice-header .id {
    color: #666;
    font-size: 14px;
}

.section {
    margin-bottom: 25px;
}

.section h3 {
    color: #444;
    margin-bottom: 15px;
    padding-bottom: 5px;
    border-bottom: 1px solid #eee;
}

.info-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
}

.info-item {
    display: flex;
    flex-direction: column;
}

.info-item .label {
    color: #666;
    font-size: 12px;
    margin-bottom: 4px;
}

.info-item .value {
    color: #333;
    font-size: 16px;
    font-weight: 500;
}

.total-box {
    background: #f8f9fa;
    padding: 20px;
    border-radius: 6px;
    margin-top: 20px;
}

.total-row {
    display: flex;
    justify-content: space-between;
    padding: 10px 0;
    border-bottom: 1px solid #dee2e6;
}

.total-row:last-child {
    border-bottom: none;
    font-weight: bold;
    font-size: 18px;
}

.no-data {
    text-align: center;
    padding: 50px;
    color: #666;
    font-style: italic;
}

.status-badge {
    display: inline-block;
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 12px;
    font-weight: bold;
    text-transform: uppercase;
}

.status-commit {
    background: #d4edda;
    color: #155724;
}

.status-default {
    background: #fff3cd;
    color: #856404;
}

.welcome-message {
    text-align: center;
    padding: 40px;
}

.welcome-message h3 {
    color: #333;
    margin-bottom: 10px;
}

.welcome-message p {
    color: #666;
}

.edit-btn, .create-new-btn {
    background: #4CAF50;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
    text-decoration: none;
    display: inline-block;
}

.edit-btn:hover, .create-new-btn:hover {
    background: #45a049;
}

.create-new-btn {
    background: #007bff;
}

.create-new-btn:hover {
    background: #0056b3;
}



/* admin */

.login-container {
    max-width: 400px;
    margin: 100px auto;
    background: white;
    border-radius: 8px;
    padding: 40px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.login-header {
    text-align: center;
    margin-bottom: 30px;
}

.login-header h1 {
    color: #333;
    font-size: 28px;
    margin-bottom: 10px;
}

.login-header p {
    color: #666;
    font-size: 14px;
}

.login-form .form-group {
    margin-bottom: 20px;
}

.login-form label {
    display: block;
    margin-bottom: 5px;
    color: #555;
    font-weight: bold;
}

.login-form input {
    width: 100%;
    padding: 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 16px;
    transition: border-color 0.3s;
}

.login-form input:focus {
    outline: none;
    border-color: #4CAF50;
}

.login-form button {
    width: 100%;
    padding: 12px;
    background: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: background 0.3s;
}

.login-form button:hover {
    background: #45a049;
}

.error-message {
    background: #f8d7da;
    color: #721c24;
    padding: 12px;
    border-radius: 4px;
    margin-bottom: 20px;
    border: 1px solid #f5c6cb;
    text-align: center;
}

.info-message {
    background: #d1ecf1;
    color: #0c5460;
    padding: 12px;
    border-radius: 4px;
    margin-bottom: 20px;
    border: 1px solid #bee5eb;
    text-align: center;
}

.back-link {
    text-align: center;
    margin-top: 20px;
}

.back-link a {
    color: #666;
    text-decoration: none;
    font-size: 14px;
}

.back-link a:hover {
    color: #333;
    text-decoration: underline;
}

.demo-credentials {
    margin-top: 30px;
    padding: 15px;
    background: #f8f9fa;
    border-radius: 4px;
    font-size: 13px;
    color: #666;
    text-align: center;
    border: 1px dashed #ddd;
}

.demo-credentials p {
    margin: 5px 0;
}

.demo-credentials strong {
    color: #333;
}

.toggle-form {
    text-align: center;
    margin-top: 20px;
    padding-top: 20px;
    border-top: 1px solid #eee;
}
.toggle-form a {
    color: #4CAF50;
    text-decoration: none;
    font-size: 14px;
}
.toggle-form a:hover {
    text-decoration: underline;
}
.form-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 20px;
}
.btn-link {
    background: none;
    border: none;
    color: #4CAF50;
    cursor: pointer;
    font-size: 14px;
    text-decoration: underline;
}
.btn-link:hover {
    color: #45a049;
}
.success-message {
    background: #d4edda;
    color: #155724;
    padding: 12px;
    border-radius: 4px;
    margin-bottom: 20px;
    border: 1px solid #c3e6cb;
    text-align: center;
}

/* edit */
.edit-form {
    background: white;
    border-radius: 8px;
    padding: 30px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.form-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    padding-bottom: 20px;
    border-bottom: 2px solid #eee;
}

.form-header h2 {
    color: #333;
}

.form-group {
    margin-bottom: 20px;
}

.form-group label {
    display: block;
    margin-bottom: 5px;
    color: #555;
    font-weight: bold;
}

.form-group input, .form-group select {
    width: 100%;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 14px;
}

.form-group input:focus, .form-group select:focus {
    outline: none;
    border-color: #4CAF50;
}

.form-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
}

.form-section {
    margin-top: 30px;
    padding-top: 20px;
    border-top: 1px solid #eee;
}

.form-section h3 {
    color: #444;
    margin-bottom: 15px;
}

.form-actions {
    display: flex;
    gap: 10px;
    margin-top: 30px;
    padding-top: 20px;
    border-top: 2px solid #eee;
}

.btn {
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    text-decoration: none;
    display: inline-block;
}

.btn-primary {
    background: #4CAF50;
    color: white;
}

.btn-primary:hover {
    background: #45a049;
}

.btn-secondary {
    background: #6c757d;
    color: white;
}

.btn-secondary:hover {
    background: #5a6268;
}

.btn-danger {
    background: #dc3545;
    color: white;
}

.btn-danger:hover {
    background: #c82333;
}

.message {
    padding: 15px;
    border-radius: 4px;
    margin-bottom: 20px;
}

.message.success {
    background: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
}

.message.error {
    background: #f8d7da;
    color: #721c24;
    border: 1px solid #f5c6cb;
}

.token-warning {
    background: #fff3cd;
    color: #856404;
    padding: 10px;
    border-radius: 4px;
    margin-bottom: 20px;
}

.reservation-selector {
    margin-bottom: 20px;
}

.reservation-selector select {
    width: 100%;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
}

/* Delete modal styles */
.modal {
    display: none;
    position: fixed;
    z-index: 1000;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.5);
}

.modal-content {
    background-color: white;
    margin: 15% auto;
    padding: 30px;
    border-radius: 8px;
    width: 400px;
    max-width: 90%;
    text-align: center;
}

.modal-content h3 {
    color: #dc3545;
    margin-bottom: 15px;
}

.modal-content p {
    margin-bottom: 20px;
    color: #666;
}

.modal-actions {
    display: flex;
    gap: 10px;
    justify-content: center;
}

.edit-btn, .create-new-btn {
    background: #4CAF50;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
    text-decoration: none;
    display: inline-block;
}

.edit-btn:hover, .create-new-btn:hover {
    background: #45a049;
}

.delete-btn {
    background: #dc3545;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
    text-decoration: none;
    display: inline-block;
    margin-left: 10px;
}

.delete-btn:hover {
    background: #c82333;
}

.button-group {
    display: flex;
    gap: 10px;
    align-items: center;
}







[Uploading Xml_Content<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<OTA_HotelResNotifRQ EchoToken="01" TimeStamp="2025-12-06T03:24:25.605" Version="1.0"
	xmlns="http://www.opentravel.org/OTA/2003/05">
	<POS>
		<Source>
		<BookingChannel Type="7">
			<CompanyName Code="traveloka">traveloka</CompanyName>
		</BookingChannel>
		</Source>
	</POS>
	<HotelReservations>
		<HotelReservation ResStatus="Commit" CreateDateTime="2025-12-06T03:24:07.483Z">
			<RoomStays>
				<RoomStay IndexNumber="1">
					<RatePlans>
						<RatePlan RatePlanCode="84068">
							<RatePlanDescription>
								<Text>Room Breakfast - INSIGHT</Text>
							</RatePlanDescription>
							<Commission>
								<CommissionableAmount CurrencyCode="IDR" Amount="123550"/>
							</Commission>
						</RatePlan>
					</RatePlans>
					<RoomRates>
						<RoomRate RoomTypeCode="20525448" NumberOfUnits="1" RatePlanCode="84068">
							<Rates>
								<Rate RoomPricingType="Per night" EffectiveDate="2026-01-31" ExpireDate="2026-02-01">
									<Base AmountBeforeTax="454664" AmountAfterTax="582449"/>
								</Rate>
							</Rates>
						</RoomRate>
					</RoomRates>
					<GuestCounts>
						<GuestCount AgeQualifyingCode="10" Count="2"/>
					</GuestCounts>
					<TimeSpan Start="2026-01-31" End="2026-02-01"/>
					<Total AmountBeforeTax="454664" AmountAfterTax="582449" CurrencyCode="IDR"/>
					<BasicPropertyInfo HotelCode="HtOPiAVZoQyJ"/>
					<ResGuestRPHs>0</ResGuestRPHs>
				</RoomStay>
			</RoomStays>
			<ResGuests>
				<ResGuest ResGuestRPH="0">
					<Profiles>
						<ProfileInfo>
							<Profile>
								<Customer>
									<PersonName>
										<GivenName>Bablu</GivenName>
										<Surname>Blaster</Surname>
									</PersonName>
									<Telephone PhoneNumber=""/>
									<Email></Email>
									<Address />
								</Customer>
							</Profile>
						</ProfileInfo>
					</Profiles>
					<SpecialRequests>
						<SpecialRequest/>
					</SpecialRequests>
				</ResGuest>
			</ResGuests>
			<ResGlobalInfo>
				<Total AmountBeforeTax="454664" AmountAfterTax="582449" CurrencyCode="IDR"/>
				<HotelReservationIDs>
					<HotelReservationID ResID_Type="13" ResID_Value="20251112009656"/>
				</HotelReservationIDs>
			</ResGlobalInfo>
		</HotelReservation>
	</HotelReservations>
</OTA_HotelResNotifRQ>.xml…]()




