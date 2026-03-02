# My-Projects  - Hotel Reservation System using XML

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
