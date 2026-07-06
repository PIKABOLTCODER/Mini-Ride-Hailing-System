# Ride-Hailing System with B+ Tree Indexing

A high-performance ride-hailing management system built in C, featuring advanced B+ tree data structures for efficient driver, passenger, and booking management.

## 📋 Overview

This system simulates a real-world ride-hailing platform (similar to Uber/Lyft) with support for:
- **Driver management** - Track active drivers with real-time locations and earnings
- **Passenger management** - Manage passenger profiles and ride frequency
- **Ride booking** - Match passengers with nearest available drivers
- **Location-based search** - Find drivers within a specified radius
- **Analytics** - Top earnings drivers, frequent passenger-driver pairs, and ride history

### Key Technology
- **B+ Tree Indexing** - O(log n) lookups for fast driver/passenger/booking retrieval
- **Real-time Geolocation** - Euclidean distance calculations for nearest driver matching
- **Data Persistence** - Automatic save/load from text files
- **Dual Vehicle Types** - Cab (₹10/km) and Bike (₹5/km) fare calculation

---

## 🏗️ Architecture

### Data Structures

#### B+ Tree Implementation
- **ORDER**: 4 (max 3 keys per node, 4 children per internal node)
- **Three separate trees**:
  1. `g_driverRoot` - Indexed by Driver ID
  2. `g_passengerRoot` - Indexed by Passenger ID
  3. `g_bookingRoot` - Indexed by Booking ID

#### Core Entities

```c
typedef struct Driver {
    int d_ID;                      // Unique driver ID
    char name[50];                 // Driver name
    int vehicle_type;              // 0=Cab, 1=Bike
    Location current_location;     // (x, y) coordinates
    int status;                    // 0=Available, 1=Booked
    float total_earnings;          // Cumulative earnings
} Driver;

typedef struct Passenger {
    int p_ID;                      // Unique passenger ID
    char name[50];                 // Passenger name
    char mobile_no[15];            // 10-digit mobile (validated)
    int frequency;                 // Number of completed rides
} Passenger;

typedef struct Booking {
    int booking_id;                // Unique booking ID
    int d_ID;                      // Driver ID
    int p_ID;                      // Passenger ID
    int vehicle_type;              // Vehicle type used
    float distance_travelled;      // Distance in km
    float fare;                    // Calculated fare
    int timestamp;                 // Unix timestamp
    int completed;                 // 0=Active, 1=Completed
} Booking;
```

### Distance Calculation
Euclidean distance formula with 5 km search radius:
```
distance = √((x₁-x₂)² + (y₁-y₂)²)
```

---

## 🚀 Getting Started

### Prerequisites
- GCC compiler
- Linux/Unix terminal or Windows with MinGW
- ~100 KB disk space

### Compilation

```bash
cd /path/to/BTREES
gcc -Wall -Wextra -o ride_hailing main.c Ride_hailing.c -lm
```

**Flags:**
- `-lm` - Link math library (required for sqrt)
- `-Wall -Wextra` - Enable all warnings (optional, for debugging)

### Running

```bash
./ride_hailing
```

On Windows:
```bash
ride_hailing.exe
```

---

## 📖 Usage Guide

### Main Menu Options

```
1. Add Driver           - Register a new driver
2. Add Passenger        - Register a new passenger
3. Request Ride         - Book a ride (matches nearest driver)
4. Complete Ride        - Mark booking as completed and calculate fare
5. Display Top Drivers  - Show top 3 drivers by earnings
6. Display Frequent Pairs - Show most frequent passenger-driver pair
7. Display Available Vehicles - List all free drivers
8. Update Driver Location - Move a driver to new coordinates
9. Delete Driver        - Remove driver from system (must be free)
10. Display Booking History - Show all ride records
11. Range Search        - Find passengers by ID range
12. Calculate Earnings  - Get total earnings for a driver
0. Exit                 - Save and close
```

### Example Workflow

#### Step 1: Add a Driver
```
Choice: 1
Driver ID: 101
Name: Raj Kumar
Vehicle Type (0=Cab, 1=Bike): 0
Current Location (x y): 10 15
→ Driver added successfully.
```

#### Step 2: Add a Passenger
```
Choice: 2
Passenger ID: 201
Name: Priya Singh
Mobile Number: 9876543210
→ Passenger added successfully.
```

#### Step 3: Request a Ride
```
Choice: 3
Passenger ID: 201
Passenger Location (x y): 12 18
Preferred Vehicle Type (-1=Any, 0=Cab, 1=Bike): 0
→ Ride booked. Booking ID: 1, Driver ID: 101 (Raj Kumar).
```

#### Step 4: Complete the Ride
```
Choice: 4
Booking ID: 1
Distance Travelled (km): 5.5
→ Ride completed for booking ID 1. Fare: 55.00
```

#### Step 5: Check Earnings
```
Choice: 12
Driver ID: 101
Total earnings of driver 101 = 55.00
```

---

## 📁 File Structure

```
BTREES/
├── README.md                    # This file
├── main.c                       # Main menu and user interface
├── Ride_hailing.h              # Header file (structs, function declarations)
├── Ride_hailing.c              # Core implementation (B+ tree, logic)
├── ride_hailing.exe            # Compiled executable
├── drivers.txt                 # Persistent driver data
├── passengers.txt              # Persistent passenger data
└── bookings.txt                # Persistent booking data
```

### Data Format

**drivers.txt** (pipe-delimited):
```
ID|Name|VehicleType|X|Y|Status|Earnings
101|Raj Kumar|0|10|15|0|55.00
```

**passengers.txt**:
```
ID|Name|Mobile|Frequency
201|Priya Singh|9876543210|1
```

**bookings.txt**:
```
BookingID|DriverID|PassengerID|VehicleType|Distance|Fare|Timestamp|Completed
1|101|201|0|5.50|55.00|1720324800|1
```

---

## ⚙️ Input Constraints

| Field | Constraint | Example |
|-------|-----------|---------|
| Driver ID | Positive integer, unique | 1-9999 |
| Passenger ID | Positive integer, unique | 1-9999 |
| Name | Max 50 characters | "Raj Kumar" |
| Mobile | Exactly 10 digits | "9876543210" |
| Vehicle Type | 0 (Cab) or 1 (Bike) | 0 |
| Location (x, y) | Non-negative integers | (10, 15) |
| Distance | Non-negative float | 5.5 |
| Preferred Type | -1 (Any), 0 (Cab), 1 (Bike) | -1 |

---

## 💰 Fare Calculation

**Formula**: `Fare = Distance × Rate`

| Vehicle | Rate | Example |
|---------|------|---------|
| Cab | ₹10/km | 5 km → ₹50 |
| Bike | ₹5/km | 5 km → ₹25 |

---

## 🔍 Search & Query Features

### Nearest Driver Matching
- Searches all available drivers (status = 0)
- Filters by vehicle type preference
- Returns driver within 5 km radius with minimum distance
- Used automatically in "Request Ride"

### Range Search (Passengers by ID)
```
Enter P_ID1 and P_ID2: 200 210
→ Returns all passengers with IDs in range [200, 210]
```

### Top Drivers Report
- Sorts all drivers by `total_earnings`
- Displays top 3 with name, ID, and earnings
- Updated after each completed ride

### Frequent Pairs Analysis
- Finds passenger-driver pair with most completed rides together
- Useful for identifying regular customers and drivers
- Uses secondary B+ tree for pair counting

---

## 🛡️ Data Integrity & Safety

### Validation Checks
✅ Duplicate ID prevention
✅ Mobile number format validation (10 digits)
✅ Location non-negative verification
✅ Distance validation (non-negative)
✅ Driver status checks (cannot update booked drivers)
✅ Booking completion flag (prevents double-completion)

### Error Handling
- Graceful null pointer checks
- Memory allocation verification
- File I/O error management
- Tree rebuild error detection

### Data Persistence
- Auto-save on every modification (add, delete, complete)
- Automatic file creation if missing
- Backwards compatible format (handles legacy bookings without `completed` field)
- Data loaded at startup

---

## 📊 Complexity Analysis

| Operation | Complexity | Notes |
|-----------|-----------|-------|
| Add Driver/Passenger | O(log n) | B+ tree insertion |
| Find Driver/Passenger | O(log n) | B+ tree search |
| Find Nearest Driver | O(n) | Linear scan needed for geo-query |
| Complete Ride | O(log n) | Tree lookup + update |
| Top Drivers | O(n log n) | Collection + sorting |
| Frequent Pairs | O(n) | Full booking scan + B+ insertion |
| Range Search | O(log n + k) | k = results count |

---

## 🐛 Troubleshooting

### Compilation Issues
```bash
# Missing math library
gcc -o ride_hailing main.c Ride_hailing.c -lm

# Path issues
cd to/project/directory && gcc ...
```

### Runtime Issues

| Issue | Solution |
|-------|----------|
| "Invalid mobile number" | Enter exactly 10 digits (0-9) |
| "Driver with ID already exists" | Use unique driver ID |
| "No free vehicle found" | Add drivers or bring them closer |
| "Cannot delete: driver is currently booked" | Wait for ride to complete |
| "Booking already completed" | Don't complete same booking twice |

### Data Issues
- Files lost? System auto-creates on startup
- Corrupted data? Delete .txt files and restart fresh
- Tree imbalance? Not possible (B+ tree is self-balancing)

---

## 🎯 Example Complete Session

```bash
$ ./ride_hailing

===== Ride-Hailing Menu =====
[...]
Enter your choice: 1
Enter Driver ID: 1
Enter Name: Alice
Enter Vehicle Type (0 = Cab, 1 = Bike): 0
Enter Current Location (x y): 0 0
→ Driver added successfully.

Enter your choice: 2
Enter Passenger ID: 1
Enter Name: Bob
Enter Mobile Number: 1234567890
→ Passenger added successfully.

Enter your choice: 3
Enter Passenger ID: 1
Enter Passenger Location (x y): 3 4
Enter Preferred Vehicle Type (-1 = Any, 0 = Cab, 1 = Bike): -1
→ Ride booked. Booking ID: 1, Driver ID: 1 (Alice).

Enter your choice: 4
Enter Booking ID: 1
Enter Distance Travelled (km): 5
→ Ride completed for booking ID 1. Fare: 50.00

Enter your choice: 12
Enter Driver ID: 1
Total earnings of driver 1 = 50.00

Enter your choice: 0
→ Exiting program.
```

---

## 📝 Notes

- All timestamps are Unix epoch (seconds since 1970-01-01)
- Locations use simple 2D Cartesian coordinates
- No authentication/security (single-user system)
- No concurrent access handling
- Max name length: 50 chars, mobile: 15 chars
- Vehicle types: only 0 (Cab) and 1 (Bike) supported

---

## 📄 License

Educational project - Free to use and modify.

---

## 👨‍💻 Author

Built as a comprehensive data structures project demonstrating:
- B+ tree implementation
- Real-world application design
- File I/O and data persistence
- Geographic search algorithms
- Memory management in C

---

## 🔄 Version History

- **v1.0** (Current) - Complete ride-hailing system with B+ tree indexing
  - Fixed booking completion tracking
  - Added real timestamp support
  - Improved error handling and validation

---

**Questions or Issues?** Review the troubleshooting section or check input constraints.
