# -------------------------------------------------------------
# File Name   : hotel_manager.py
# Author      : Rohit Sharma
# Description : Hotel Management System for managing room booking, allocation, and checkout.
# Date        : 20 June 2025
# -------------------------------------------------------------

###### from datetime import datetime, timedelta
import os
from datetime import datetime  # Importing datetime module to handle date and time operations
from datetime import datetime, timedelta  # Add timedelta to the import

class HotelManager:
    def __init__(self):
        self.rooms = {}  # Available rooms: room_number -> {'type': str, 'price': float, 'capacity': int}
        self.booked = {}  # Booked rooms: room_number -> {'guest': str, 'check_in': datetime, 'check_out': datetime}
        self.file_name = "LHMS_850003440.txt"
        self.backup_template = "850003440_Backup_{}.txt"
        self.load_rooms()  # Load initial room data
        self.load_booked()  # Load booking records if they exist

    def load_rooms(self):
        """Load initial room data into the system"""
        # Sample initial rooms
        initial_rooms = {
            '101': {'type': 'Standard', 'price': 100.00, 'capacity': 2},
            '102': {'type': 'Standard', 'price': 100.00, 'capacity': 2},
            '201': {'type': 'Deluxe', 'price': 150.00, 'capacity': 3},
            '202': {'type': 'Deluxe', 'price': 150.00, 'capacity': 3},
            '301': {'type': 'Suite', 'price': 250.00, 'capacity': 4}
        }
        self.rooms.update(initial_rooms)

    def load_booked(self):
        """Load booking records from file if it exists"""
        try:
            if os.path.exists(self.file_name):
                with open(self.file_name, 'r') as f:
                    for line in f:
                        room_num, guest, check_in_str, check_out_str = line.strip().split(',')
                        check_in = datetime.strptime(check_in_str, '%Y-%m-%d')
                        check_out = datetime.strptime(check_out_str, '%Y-%m-%d')
                        self.booked[room_num] = {
                            'guest': guest,
                            'check_in': check_in,
                            'check_out': check_out
                        }
        except Exception as e:
            print(f"Error loading booking records: {e}")

    def add_room(self, room_number, room_type, price, capacity):
        """Add a new room to the hotel"""
        if room_number in self.rooms:
            print("Room number already exists.")
        else:
            self.rooms[room_number] = {
                'type': room_type,
                'price': float(price),
                'capacity': int(capacity)
            }
            print(f"Room {room_number} ({room_type}) added successfully.")

    def remove_room(self, room_number):
        """Remove a room from the hotel"""
        if room_number in self.rooms:
            if room_number in self.booked:
                print("Cannot remove a room that is currently booked.")
            else:
                removed = self.rooms.pop(room_number)
                print(f"Room {room_number} ({removed['type']}) removed successfully.")
        else:
            print("Room number not found.")

    def display_rooms(self):
        """Display all available rooms"""
        if not self.rooms:
            print("No rooms available.")
        else:
            print("\nAvailable Rooms:")
            print("{:<10} {:<15} {:<10} {:<10} {:<15}".format(
                "Room No.", "Type", "Price", "Capacity", "Status"))
            print("-" * 60)
            
            for room_num, details in self.rooms.items():
                status = "Booked" if room_num in self.booked else "Available"
                print("{:<10} {:<15} ${:<9.2f} {:<10} {:<15}".format(
                    room_num, details['type'], details['price'], 
                    details['capacity'], status))

    def book_room(self, room_number, guest_name, nights):
        """Book a room for a guest"""
        if room_number not in self.rooms:
            print("Room number not found.")
        elif room_number in self.booked:
            print("Room is already booked.")
        else:
            check_in = datetime.now()
            check_out = check_in + timedelta(days=nights)
            total_price = self.rooms[room_number]['price'] * nights
            
            self.booked[room_number] = {
                'guest': guest_name,
                'check_in': check_in,
                'check_out': check_out,
                'total_price': total_price
            }
            
            print(f"\nRoom {room_number} booked successfully for {guest_name}")
            print(f"Check-in: {check_in.strftime('%Y-%m-%d')}")
            print(f"Check-out: {check_out.strftime('%Y-%m-%d')}")
            print(f"Total price: ${total_price:.2f}")

    def view_bookings(self):
        """View all current bookings"""
        if not self.booked:
            print("No rooms are currently booked.")
        else:
            print("\nCurrent Bookings:")
            print("{:<10} {:<20} {:<15} {:<15} {:<10}".format(
                "Room No.", "Guest", "Check-in", "Check-out", "Price"))
            print("-" * 70)
            
            for room_num, details in self.booked.items():
                print("{:<10} {:<20} {:<15} {:<15} ${:<9.2f}".format(
                    room_num, 
                    details['guest'],
                    details['check_in'].strftime('%Y-%m-%d'),
                    details['check_out'].strftime('%Y-%m-%d'),
                    details['total_price']))

    def check_out(self, room_number):
        """Check out a guest and calculate any additional charges"""
        if room_number not in self.booked:
            print("Room is not currently booked.")
        else:
            booking = self.booked.pop(room_number)
            actual_check_out = datetime.now()
            
            # Calculate if early or late checkout
            original_check_out = booking['check_out']
            days_diff = (actual_check_out - original_check_out).days
            
            if days_diff > 0:
                # Late checkout - charge for extra days
                extra_charge = days_diff * self.rooms[room_number]['price']
                print(f"Late checkout! Additional charge: ${extra_charge:.2f}")
            elif days_diff < 0:
                # Early checkout - possible refund
                refund = abs(days_diff) * self.rooms[room_number]['price'] * 0.5
                print(f"Early checkout! Partial refund: ${refund:.2f}")
            
            print(f"{booking['guest']} has checked out of room {room_number}.")

    def save_bookings_to_file(self):
        """Save current bookings to file"""
        try:
            with open(self.file_name, "w") as f:
                for room_num, details in self.booked.items():
                    line = f"{room_num},{details['guest']},{details['check_in'].strftime('%Y-%m-%d')},{details['check_out'].strftime('%Y-%m-%d')}\n"
                    f.write(line)
            print("Booking records saved successfully.")
        except Exception as e:
            print(f"Error saving bookings: {e}")

    def display_bookings_file(self):
        """Display contents of bookings file"""
        try:
            with open(self.file_name, "r") as f:
                print("\nBookings File Contents:")
                print(f.read())
        except FileNotFoundError:
            print("Bookings file not found.")
   
    def backup_and_clear_file(self):
        """Create backup of bookings file and clear it"""
        try:
            if os.path.exists(self.file_name):
                timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
                backup_file = self.backup_template.format(timestamp)
                
                with open(self.file_name, "r") as original, open(backup_file, "w") as backup:
                    backup.write(original.read())
                
                open(self.file_name, "w").close()
                print(f"Backup created at '{backup_file}' and file cleared.")
            else:
                print("No bookings file to backup.")
        except Exception as e:
            print(f"Error during backup: {e}")
    
    """
The class room manager will hold the functions that the program will be using such as:
__init__
add_room
delete_room
display_rooms
allocate_room
view_allocated_rooms
de_allocate_room
save_allocated_to_file
display_file
backup_and_clear_file
"""

if __name__ == "__main__":
    hotel = HotelManager()
    
    while True:
        print("\nHotel Management System")
        print("1. Add Room")
        print("2. Remove Room")
        print("3. Display Rooms")
        print("4. Book Room")
        print("5. View Bookings")
        print("6. Check Out")
        print("7. Save Bookings to File")
        print("8. Display Bookings File")
        print("9. Backup and Clear File")
        print("0. Exit")
        
        choice = input("Enter your choice: ")
        
        if choice == '1':
            room_num = input("Enter room number: ")
            room_type = input("Enter room type: ")
            price = float(input("Enter price per night: "))
            capacity = int(input("Enter room capacity: "))
            hotel.add_room(room_num, room_type, price, capacity)
        
        elif choice == '2':
            room_num = input("Enter room number to remove: ")
            hotel.remove_room(room_num)
        
        elif choice == '3':
            hotel.display_rooms()
        
        elif choice == '4':
            room_num = input("Enter room number to book: ")
            guest = input("Enter guest name: ")
            nights = int(input("Enter number of nights: "))
            hotel.book_room(room_num, guest, nights)
        
        elif choice == '5':
            hotel.view_bookings()
        
        elif choice == '6':
            room_num = input("Enter room number to check out: ")
            hotel.check_out(room_num)
        
        elif choice == '7':
            hotel.save_bookings_to_file()
        
        elif choice == '8':
            hotel.display_bookings_file()
        
        elif choice == '9':
            hotel.backup_and_clear_file()
        
        elif choice == '0':
            print("Exiting Hotel Management System.")
            break
        
        else:
            print("Invalid choice. Please try again.")
def save_allocations_to_file(self):
    """Save current room allocations to LHMS_StudentID.txt file (Menu Option 7)"""
    try:
        filename = f"LHMS_{self,.850003440}.txt"
        
        with open(filename, 'w') as file:
            # Write header
            file.write("Hotel Room Allocation Records\n")
            file.write("="*50 + "\n")
            file.write(f"Generated on: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n\n")
            
            # Write room allocation data
            if not self.booked:
                file.write("No rooms currently allocated.\n")
            else:
                file.write("{:<10} {:<20} {:<15} {:<15} {:<10}\n".format(
                    "Room No.", "Guest Name", "Check-In", "Check-Out", "Amount"))
                file.write("-"*70 + "\n")
                
                for room_num, details in self.booked.items():
                    file.write("{:<10} {:<20} {:<15} {:<15} ${:<9.2f}\n".format(
                        room_num,
                        details['guest'],
                        details['check_in'].strftime('%Y-%m-%d'),
                        details['check_out'].strftime('%Y-%m-%d'),
                        details['total_price']))
            
            # Write footer
            file.write("\n" + "="*50 + "\n")
            file.write(f"Total allocated rooms: {len(self.booked)}\n")
        
        print(f"\nRoom allocations successfully saved to {filename}")
        return True
        
    except Exception as e:
        print(f"\nError saving to file: {str(e)}")
        return False
    
