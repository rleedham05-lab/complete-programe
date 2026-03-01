# complete-programe
#this is the final program including encryption , projetile motion sym, casuilty calculator sym, and password reset
# importing all packages 
import math
import time
import os
import sys
import numpy as np
import tkinter as tk
from tkinter import *
from matplotlib.figure import Figure
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
from tkinter import messagebox
from math import sqrt  
import matplotlib.pyplot as plt
import hashlib
from hashlib import sha256 

# checks if the file exists 
def check_file():
    return os.path.exists("password.txt")

#writable file 
def writing_Password_new(encrypted_password):
    with open("password.txt", "w") as f:
        f.write(encrypted_password)

#encryption system 
def password_encryption(password):
    encrypted_password = (sha256(password.encode('utf-8')).hexdigest())
    return encrypted_password


font1 = {'family': 'sans-serif', 'color': 'grey', 'size': 15}
font2 = {'family': 'sans-serif', 'color': 'grey', 'size': 30}

#check to see if file is empty 
file_exists = check_file()
if not file_exists:
    password = "Password"
    encrypted_password = password_encryption(password)
    writing_Password_new(encrypted_password)

#timeout feature
def time_out(callback):
    global login_window
    t = 30
    messagebox.showwarning("Time Out Error", "Please wait 30 seconds before being allowed to try again")
    if login_window is not None:
        login_window.after(t * 1000, callback)
    else:
        print("Warning: login_window is None, cannot set timeout")  

#subroutine for incorrect password 
def incorrect_password(user_password, attempts):
    messagebox.showwarning("Incorrect", f"{user_password} is not the password. You have {attempts} attempts remaining.")
    return attempts - 1

#retrieves the stored password 
def get_stored_password():
    if os.path.exists("password.txt"):
        with open("password.txt", "r") as f:
            return f.read().strip()
    return None

attempts_remaining = 3
#setting global variables of the program to none 
login_button = None
entry_label = None
login_window = None 

#password verification 
def verify_password():
    global attempts_remaining, login_button, entry_label, login_window
    # Get user input from entry 
    user_password = entry_label.get()
    user_encrypted_password = password_encryption(user_password)
    stored_password = get_stored_password()
    
    if user_encrypted_password == stored_password:
        messagebox.showinfo("Success", "Login successful!")
        login_window.destroy() 
        launch_main_application()
    else:
        attempts_remaining = incorrect_password(user_password, attempts_remaining)
        entry_label.delete(0, END)  
        
        if attempts_remaining <= 0:
            login_button.config(state='disabled')
            
            def enable_button():
                global attempts_remaining
                attempts_remaining = 3
                login_button.config(state='normal')
            
            time_out(enable_button)

def launch_main_application():
    #initial design layout for gui system 
    main_window = tk.Tk()
    main_window.geometry("1600x900")
    main_window.title("Projectile Motion Simulator  Main Application")
    main_window.configure(bg='black')
    Label(main_window, text="Projectile Motion Simulator", font=font2, bg='black', fg='white').pack(pady=50)
    Label(main_window, text="(Welcome to projectile motion sim and casuilty calculation sim)", font=font1, bg='black', fg='grey').pack()
    Label(main_window, text="please select function", font=font2, bg='black', fg='grey').pack()
    
    password_change = Button(main_window, text="change password:", command=reset_password, width=15)
    continue_button = Button(main_window, text="proceed to projectile motion sym:", command=lambda: [main_window.destroy(), projectile_simulations()], width=25)
    casuilty_calculation_system_button = Button(main_window, text="proceed to casuilty calculation system", command=lambda: [main_window.destroy(), Casuilty_calculation()], width=25)
    exit_button = Button(main_window, text="Exit program:", command=main_window.destroy, width=15)
    
    password_change.pack(pady=10)
    continue_button.pack(pady=10)
    casuilty_calculation_system_button.pack(pady=10)
    exit_button.pack(pady=10)
    
    main_window.mainloop()

#launch projectile motion simulator 
def projectile_simulations():
    # Global variables defined 
    entry_velocity = None
    entry_mass = None
    entry_angle = None
    entry_emistivity = None
    entry_specific_heat_capacity = None
    entry_area = None
    entry_density = None
    entry_drag_coeffecient = None
    entry_enviroment_temperature = None

    # Global variables labels
    velocity_Label = None
    mass_Label = None
    angle_Label = None
    emistivity_Label = None
    specific_heat_capacity_Label = None
    aera_Label = None
    density_Label = None
    drag_coeffecient = None
    enviroment_temperature = None
    Fdrag_Label = None
    T_label = None
    t_Label = None


    # checks value is a compatible float
    def check_value_float(value):
        try:
            value = float(value)
            return True 
        except ValueError:
            return False 
        
    # calculating value for atmospheric temperature scalar
    def atmosphere_temp(E, y_displacement):
        factor = 9.8/1000
        E = E - (y_displacement * factor)
        return E 

    # finding the value of Force drag
    def calculations_Fdrag(density, velocity, area, Cd):
        p = density
        v = velocity
        A = area
        if abs(v) > 10000:
            v = 10000 if v > 0 else -10000
        Fdrag = (p * (v**2) * A * Cd) / 2
        if Fdrag > 1e10:
            Fdrag = 1e10
        return Fdrag

    # value finding for the value of calculations of deceleration and new value of v in both x and y axis 
    def calculaion_rate_deceleration(m, x_velocity, y_velocity, velocity, g, time_period, Fdrag):
        vx = x_velocity
        vy = y_velocity
        v = velocity 
        
        if abs(v) < 0.001:
            v = 0.001
        
        if m < 0.001:
            m = 0.001
        
        drag_decel_x = (-Fdrag * (vx / v)) / m
        drag_decel_y = (-Fdrag * (vy / v)) / m
        
        deceleration_x = drag_decel_x
        deceleration_y = drag_decel_y - g
        
        if abs(deceleration_x) > 1000:
            deceleration_x = 1000 if deceleration_x > 0 else -1000
        if abs(deceleration_y) > 1000:
            deceleration_y = 1000 if deceleration_y > 0 else -1000
        
        new_vx = vx + (deceleration_x * time_period)
        new_vy = vy + (deceleration_y * time_period)
        
        if abs(new_vx) > 10000:
            new_vx = 10000 if new_vx > 0 else -10000
        if abs(new_vy) > 10000:
            new_vy = 10000 if new_vy > 0 else -10000
        
        new_v_value = sqrt((new_vx**2) + (new_vy**2))
        
        if new_v_value > 10000:
            new_v_value = 10000
        
        return new_v_value, new_vx, new_vy

    # calculate temperature change from drag heating and radiation cooling
    def calculate_temperature_change(T_object, Fdrag, velocity, time_period, m, c, e, sigma, A, E):
        if m < 0.001:
            m = 0.001
        if c < 0.001:
            c = 0.001
        
        # heat generated by drag
        heat_generated = Fdrag * velocity * time_period
        
        if heat_generated > 1e8:
            heat_generated = 1e8
        
        # heat radiated using stefan boltzmann
        if T_object > 0:
            temp_diff_4 = T_object**4 - E**4
            if abs(temp_diff_4) > 1e15:
                temp_diff_4 = 1e15 if temp_diff_4 > 0 else -1e15
            heat_radiated = e * sigma * A * temp_diff_4 * time_period
        else:
            heat_radiated = 0
        
        if abs(heat_radiated) > 1e8:
            heat_radiated = 1e8 if heat_radiated > 0 else -1e8
        
        net_heat = heat_generated - heat_radiated
        
        delta_T = net_heat / (m * c)
        
        if abs(delta_T) > 100:
            delta_T = 100 if delta_T > 0 else -100
        
        T_new = T_object + delta_T
        
        if T_new < 0:
            T_new = 0
        
        if T_new > 10000:
            T_new = 10000
        
        return T_new

    # subroutine to calculate velocity in x 
    def calculations_velocity_x(velocity, a):
        v = velocity 
        r_a = np.radians(a)   
        v_x = velocity * math.cos(r_a)
        return v_x

    # subroutine to calculate velocity in y  
    def calculations_velocity_y(velocity, a):
        v = velocity 
        r_a = np.radians(a) 
        v_y = velocity * math.sin(r_a)
        return v_y

    # checks when the value for Y_max is achieved 
    def check_Y_max(velocity_y, time_period, current_displacement_y):
        V_y = velocity_y
        t = time_period
        D_y = current_displacement_y
        if V_y <= 0:
            Y_max = D_y
            return True
        else:
            return False

    # checks an iteration loop to find the range 
    def find_range(displacement_y, displacement_x):
        d_y = displacement_y
        d_x = displacement_x
        if d_y <= 0:
            range_object = d_x
            return range_object
        else:
            return d_x

    # error display panel
    def error_numeric():
        messagebox.showerror("Incorrect Error", "Error: please enter valid number")

    # collecting variables from gui 

    # collecting velocity value 
    def my_function_v():
        nonlocal entry_velocity
        value = entry_velocity.get()
        correct = check_value_float(value)
        if correct == True:
            v = value
            return float(v)
        else:
            return False 

    # collecting mass value 
    def my_function_m():
        nonlocal entry_mass
        value = entry_mass.get()
        correct = check_value_float(value)
        if correct == True:
            m = value
            return float(m)
        else:
            return False 

    # collecting angle value 
    def my_function_a():
        nonlocal entry_angle
        value = entry_angle.get()
        correct = check_value_float(value)
        if correct == True:
            a = value
            return float(a)
        else:
            return False
        
    # collecting emissivity value 
    def my_function_e():
        nonlocal entry_emistivity
        value = entry_emistivity.get()
        correct = check_value_float(value)
        if correct == True:
            e = value
            return float(e)
        else:
            return False 
        
    # collecting specific heat value 
    def my_function_c():
        nonlocal entry_specific_heat_capacity
        value = entry_specific_heat_capacity.get()
        correct = check_value_float(value)
        if correct == True:
            c = value
            return float(c)
        else:
            return False 
        
    # collecting area value 
    def my_function_A():
        nonlocal entry_area
        value = entry_area.get()
        correct = check_value_float(value)
        if correct == True:
            A = value
            return float(A)
        else:
            return False 

    # collecting density value 
    def my_function_p():
        nonlocal entry_density
        value = entry_density.get()
        correct = check_value_float(value)
        if correct == True:
            p = value
            return float(p)
        else:
            return False 
        
    # collecting drag coefficient 
    def my_function_Cd():
        nonlocal entry_drag_coeffecient
        value = entry_drag_coeffecient.get()
        correct = check_value_float(value)
        if correct == True:
            Cd = value
            return float(Cd)
        else:
            return False   

    def my_function_E():
        nonlocal entry_enviroment_temperature
        value = entry_enviroment_temperature.get()
        correct = check_value_float(value)
        if correct == True:
            E = value
            return float(E)
        else:
            return False   
        
    # update subroutine for gui  
    # change subroutine for the velocity value
    def change_v(v, velocity_Label):
        velocity_Label.config(text="velocity " + str(v) + " ms^-1")

    # change subroutine for the mass value
    def change_m(m, mass_Label):
        mass_Label.config(text="mass " + str(m) + " kg")

    # change subroutine for the angle value
    def change_a(a, angle_label):
        angle_label.config(text="angle " + str(a) + " degrees")

    # change subroutine for the emissivity 
    def change_e(e, emistivity_Label):
        emistivity_Label.config(text="emissivity " + str(e))

    # change specific heat capacity 
    def change_c(c, specific_heat_capacity_Label):
        specific_heat_capacity_Label.config(text="specific heat capacity " + str(c) + "")

    # change area
    def change_A(A, aera_Label):
        aera_Label.config(text="area " + str(A) + " m^2")

    # change density 
    def change_p(p, density_Label):
        density_Label.config(text="density " + str(p) + " kg/m^3")

    # change Fdrag value
    def change_Fdrag(Fdrag, Fdrag_Label):
        Fdrag_Label.config(text="force in drags is " + str(Fdrag) + " N")

    # change Temperature value 
    def change_T(T, T_label):
        T_label.config(text="Temperature of the object is " + str(T) + " K")

    # change environment value 
    def change_E(E, enviroment_temperature_label):
        enviroment_temperature_label.config(text="temperature of environment is " + str(E) + " K")

    # change time value 
    def change_t(t, t_Label):
        t_Label.config(text="time period of the object is " + str(t) + " seconds")

    def value_set_up(root):
        nonlocal entry_velocity, entry_mass, entry_angle, entry_emistivity
        nonlocal entry_specific_heat_capacity, entry_area, entry_density
        nonlocal entry_drag_coeffecient, entry_enviroment_temperature
        nonlocal velocity_Label, mass_Label, angle_Label, emistivity_Label
        nonlocal specific_heat_capacity_Label, aera_Label, density_Label
        nonlocal drag_coeffecient, enviroment_temperature
        
        # Setting up labels of inputted variables
        title_label = Label(root, text="Variables")  
        velocity_Label = Label(root, text="velocity: not set")
        mass_Label = Label(root, text="mass: not set")  
        angle_Label = Label(root, text="angle: not set")
        emistivity_Label = Label(root, text="emissivity: not set")
        specific_heat_capacity_Label = Label(root, text="specific heat capacity: not set")
        aera_Label = Label(root, text="area: not set")
        density_Label = Label(root, text="density: not set")
        drag_coeffecient = Label(root, text="drag coefficient: not set")
        enviroment_temperature = Label(root, text="temperature of your environment: not set")
        
        # Displaying labels
        title_label.grid(row=1, column=0)
        velocity_Label.grid(row=2, column=0)
        mass_Label.grid(row=3, column=0)
        angle_Label.grid(row=4, column=0)
        emistivity_Label.grid(row=5, column=0)
        specific_heat_capacity_Label.grid(row=6, column=0)
        aera_Label.grid(row=7, column=0)
        density_Label.grid(row=8, column=0)
        drag_coeffecient.grid(row=9, column=0)
        enviroment_temperature.grid(row=10, column=0)
        
        # Setting up response zones 
        entry_velocity = tk.Entry(root)
        entry_velocity.grid(row=2, column=1)
        entry_mass = tk.Entry(root)
        entry_mass.grid(row=3, column=1)
        entry_angle = tk.Entry(root)
        entry_angle.grid(row=4, column=1)
        entry_emistivity = tk.Entry(root)
        entry_emistivity.grid(row=5, column=1)
        entry_specific_heat_capacity = tk.Entry(root)
        entry_specific_heat_capacity.grid(row=6, column=1)
        entry_area = tk.Entry(root)
        entry_area.grid(row=7, column=1)
        entry_density = tk.Entry(root)
        entry_density.grid(row=8, column=1)
        entry_drag_coeffecient = tk.Entry(root)
        entry_drag_coeffecient.grid(row=9, column=1)
        entry_enviroment_temperature = tk.Entry(root)
        entry_enviroment_temperature.grid(row=10, column=1)
        
        # Setting up buttons for entering values 
        
        # Entry for velocity 
        def velocity_entered():
            result = my_function_v()
            if result == False:
                error_numeric()
            else:
                change_v(result, velocity_Label)
        
        velocity_button = tk.Button(root, text="Enter", command=velocity_entered)
        velocity_button.grid(row=2, column=2)
        
        # Entry for mass
        def mass_entered():
            result = my_function_m()
            if result == False:
                error_numeric()
            else:
                change_m(result, mass_Label)
        
        mass_button = tk.Button(root, text="Enter", command=mass_entered)
        mass_button.grid(row=3, column=2)

        # Entry for angle 
        def angle_entered():
            result = my_function_a()
            if result == False:
                error_numeric()
            else:
                change_a(result, angle_Label)
        
        angle_button = tk.Button(root, text="Enter", command=angle_entered)
        angle_button.grid(row=4, column=2)
        
        # Entry for emissivity    
        def emissivity_entered():
            result = my_function_e()
            if result == False:
                error_numeric()
            else:
                change_e(result, emistivity_Label)
        
        emistivity_button = tk.Button(root, text="Enter", command=emissivity_entered)
        emistivity_button.grid(row=5, column=2)
        
        # Entry for specific heat capacity 
        def specific_heat_entered():
            result = my_function_c()
            if result == False:
                error_numeric()
            else:
                change_c(result, specific_heat_capacity_Label)
        
        specific_heat_button = tk.Button(root, text="Enter", command=specific_heat_entered)
        specific_heat_button.grid(row=6, column=2)
            
        # Entry for area 
        def area_entered():
            result = my_function_A()
            if result == False:
                error_numeric()
            else:
                change_A(result, aera_Label)
        
        area_button = tk.Button(root, text="Enter", command=area_entered)
        area_button.grid(row=7, column=2)
            
        # Entry for density
        def density_entered():
            result = my_function_p()
            if result == False:
                error_numeric()
            else:
                change_p(result, density_Label)
        
        density_button = tk.Button(root, text="Enter", command=density_entered)
        density_button.grid(row=8, column=2)
            
        # Entry for drag coefficient
        def drag_coeff_entered():
            result = my_function_Cd()
            if result == False:
                error_numeric()
            else:
                drag_coeffecient.config(text="drag coefficient " + str(result))
        
        drag_coeffecient_button = tk.Button(root, text="Enter", command=drag_coeff_entered)
        drag_coeffecient_button.grid(row=9, column=2)
        
        # Entry for environmental temperature 
        def env_temp_entered():
            result = my_function_E()
            if result == False:
                error_numeric()
            else:
                change_E(result, enviroment_temperature)
        
        enviroment_temperature_button = tk.Button(root, text="Enter", command=env_temp_entered)
        enviroment_temperature_button.grid(row=10, column=2)

    def calculated_value_set_up(root):
        nonlocal Fdrag_Label, T_label, t_Label
        
        # Setting up labels 
        Fdrag_Label = Label(root, text="force in drags is: not calculated")
        T_label = Label(root, text="Temperature of the object is: not calculated")
        t_Label = Label(root, text="time period of the object is: not calculated")
        
        # Displaying labels 
        Fdrag_Label.grid(row=11, column=0)
        T_label.grid(row=12, column=0)
        t_Label.grid(row=13, column=0)

    # Iteration subroutine
    def calculation_iteration_subroutine(a, m, e, c, p, v, A, Cd, E, 
                                        ax1, ax2, ax3, ax4, ax5, ax6, 
                                        fig1, fig2, fig3, fig4, fig5, fig6,
                                        canvas1, canvas2, canvas3, canvas4, canvas5, canvas6, root):
        
        if v <= 0 or v > 10000:
            messagebox.showerror("Invalid Input", "Velocity must be between 0 and 10000 m/s")
            return
        if m <= 0 or m > 1e6:
            messagebox.showerror("Invalid Input", "Mass must be between 0 and 1,000,000 kg")
            return
        if a < 0 or a > 90:
            messagebox.showerror("Invalid Input", "Angle must be between 0 and 90 degrees")
            return
        if p <= 0 or p > 10:
            messagebox.showerror("Invalid Input", "Density must be between 0 and 10 kg/m³")
            return
        
        # Lists to store data for plotting
        x_positions = [0]
        y_positions = [0]
        time_values = [0]
        drag_values = [0]
        temperature_values = [E]
        velocity_values = [v]
        x_velocity_values = []
        y_velocity_values = []
        
        # Calculate initial velocity components ONCE at the start
        velocity = v
        x_velocity = calculations_velocity_x(v, a)
        y_velocity = calculations_velocity_y(v, a)
        
        x_velocity_values.append(x_velocity)
        y_velocity_values.append(y_velocity)
        
        x_displacement = 0
        y_displacement = 0
        
        t = 0
        time_period = 0.01
        g = 9.81
        sigma = 5.67e-8
        
        T_object = E
        
        max_iterations = 100000
        iteration_count = 0
        
        try:
            while y_displacement > -1 and iteration_count < max_iterations:
                iteration_count = iteration_count + 1
                
                x_displacement_new = x_velocity * time_period
                y_displacement_new = y_velocity * time_period
                
                if abs(x_displacement_new) > 1e6 or abs(y_displacement_new) > 1e6:
                    messagebox.showwarning("Simulation Stopped", "Displacement values too large - simulation stopped")
                    break
                
                x_displacement = x_displacement + x_displacement_new
                y_displacement = y_displacement + y_displacement_new
                
                if y_displacement < 0:
                    break
                
                Fdrag = calculations_Fdrag(p, velocity, A, Cd) 
                
                if not np.isfinite(Fdrag):
                    messagebox.showerror("Calculation Error", "Invalid drag force calculated")
                    break
                
                T_object = calculate_temperature_change(T_object, Fdrag, velocity, time_period, m, c, e, sigma, A, E)
                
                if not np.isfinite(T_object):
                    T_object = E
                
                new_velocity, new_x_velocity, new_y_velocity = calculaion_rate_deceleration(
                    m, x_velocity, y_velocity, velocity, g, time_period, Fdrag)
                
                if not np.isfinite(new_velocity):
                    messagebox.showerror("Calculation Error", "Invalid velocity calculated")
                    break
                
                velocity = new_velocity
                x_velocity = new_x_velocity
                y_velocity = new_y_velocity
                
                x_positions.append(x_displacement)
                y_positions.append(y_displacement)
                time_values.append(t)
                drag_values.append(Fdrag)
                temperature_values.append(T_object)
                velocity_values.append(velocity)
                x_velocity_values.append(x_velocity)
                y_velocity_values.append(y_velocity)
                
                # Update plots every 10 iterations for performance
                if len(time_values) % 10 == 0:
                    # Plot 1: Trajectory
                    ax1.clear()
                    ax1.plot(x_positions, y_positions, 'b-')
                    ax1.set_title("Displacement over time")
                    ax1.set_xlabel("Range (m)")
                    ax1.set_ylabel("Height (m)")
                    ax1.grid(True)
                    fig1.tight_layout()
                    
                    # Plot 2: Thermal energy
                    ax2.clear()
                    ax2.plot(time_values, temperature_values, 'r-')
                    ax2.set_title("Object temperature")
                    ax2.set_xlabel("Time (s)")
                    ax2.set_ylabel("Temperature (K)")
                    ax2.grid(True)
                    fig2.tight_layout()
                    
                    # Plot 3: Drag force
                    ax3.clear()
                    ax3.plot(time_values, drag_values, 'g-')
                    ax3.set_title("Air resistance/drag over time")
                    ax3.set_xlabel("Time (s)")
                    ax3.set_ylabel("Drag force (N)")
                    ax3.grid(True)
                    fig3.tight_layout()
                    
                    # Plot 4: Y velocity
                    ax4.clear()
                    ax4.plot(time_values, y_velocity_values, 'purple')
                    ax4.set_title("Velocity in y against time")
                    ax4.set_xlabel("Time (s)")
                    ax4.set_ylabel("Velocity in Y (m/s)")
                    ax4.grid(True)
                    fig4.tight_layout()
                    
                    # Plot 5: X velocity
                    ax5.clear()
                    ax5.plot(time_values, x_velocity_values, 'orange')
                    ax5.set_title("Velocity in x against time")
                    ax5.set_xlabel("Time (s)")
                    ax5.set_ylabel("Velocity in X (m/s)")
                    ax5.grid(True)
                    fig5.tight_layout()
                    
                    # Plot 6: Total velocity
                    ax6.clear()
                    ax6.plot(time_values, velocity_values, 'cyan')
                    ax6.set_title("Velocity against time")
                    ax6.set_xlabel("Time (s)")
                    ax6.set_ylabel("Velocity (m/s)")
                    ax6.grid(True)
                    fig6.tight_layout()
                    
                    canvas1.draw()
                    canvas2.draw()
                    canvas3.draw()
                    canvas4.draw()
                    canvas5.draw()
                    canvas6.draw()
                    root.update()
                    
                    change_Fdrag(Fdrag, Fdrag_Label)
                    change_t(t, t_Label)
                    change_T(T_object, T_label)
                
                t = t + time_period
                
                if t > 1000:
                    messagebox.showinfo("Simulation Complete", "Maximum simulation time (1000s) reached")
                    break
            
            if iteration_count >= max_iterations:
                messagebox.showwarning("Simulation Stopped", "Maximum iterations reached")
            
            ax1.clear()
            ax1.plot(x_positions, y_positions, 'b-')
            ax1.set_title("Displacement over time")
            ax1.set_xlabel("Range (m)")
            ax1.set_ylabel("Height (m)")
            ax1.grid(True)
            fig1.tight_layout()
            canvas1.draw()
            
            max_height = max(y_positions) if y_positions else 0
            final_temp = T_object
            messagebox.showinfo("Simulation Complete",
                            "Simulation finished!\nRange: " + str(x_displacement) + " m\nMax Height: " + str(max_height) + " m\nTime: " + str(t) + " s\nFinal Temperature: " + str(final_temp) + " K")
        
        except Exception as ex:
            messagebox.showerror("Calculation Error", "An error occurred during calculation:\n" + str(ex))
            return

    # Check values of entry before calculation 
    def Check_Values(a, m, e, c, p, v, A, Cd, E):
        start = False
        total_correct = 0
        if a != 0:
            total_correct = total_correct + 1
        if m != 0:
            total_correct = total_correct + 1
        if e != 0:
            total_correct = total_correct + 1
        if c != 0:
            total_correct = total_correct + 1
        if p != 0:
            total_correct = total_correct + 1
        if v != 0:
            total_correct = total_correct + 1
        if A != 0:
            total_correct = total_correct + 1
        if Cd != 0:
            total_correct = total_correct + 1
        if E != 0:
            total_correct = total_correct + 1
        if total_correct == 9:
            start = True
        else:
            start = False 
        return start 

    # Main execution 
    def main():
        # Setting up backdrop 
        root = tk.Tk()
        root.geometry("1600x900")
        root.title("Projectile Motion Simulator")
        root.configure(bg='black')
        
        # Font library 
        font1 = {'family': 'sans-serif', 'color': 'black', 'size': 15}
        font2 = {'family': 'sans-serif', 'color': 'black', 'size': 10}  

        # Create first figure Displacement (top-left)
        fig1 = Figure(figsize=(4, 2.5))
        fig1.patch.set_facecolor('grey')
        ax1 = fig1.add_subplot(111)
        ax1.set_title("Displacement over time")
        ax1.set_xlabel("Range (m)")
        ax1.set_ylabel("Height (m)")
        ax1.plot(0, 0)
        ax1.grid(True)
        fig1.tight_layout()
        
        canvas1 = FigureCanvasTkAgg(fig1, master=root)
        canvas1.draw()
        canvas1.get_tk_widget().grid(row=0, column=3, rowspan=5, columnspan=2, sticky='nsew', padx=5, pady=5)
        
        # Create second figure Thermal energy (top-middle)
        fig2 = Figure(figsize=(4, 2.5))
        fig2.patch.set_facecolor('grey')
        ax2 = fig2.add_subplot(111)
        ax2.set_title("Object temperature")
        ax2.set_xlabel("Time (s)")
        ax2.set_ylabel("Temperature (K)")
        ax2.plot(0, 0)
        ax2.grid(True)
        fig2.tight_layout()
        
        canvas2 = FigureCanvasTkAgg(fig2, master=root)
        canvas2.draw()
        canvas2.get_tk_widget().grid(row=0, column=5, rowspan=5, columnspan=2, sticky='nsew', padx=5, pady=5)
        
        # Create third figure air resistance (top-right)
        fig3 = Figure(figsize=(4, 2.5))
        fig3.patch.set_facecolor('grey')
        ax3 = fig3.add_subplot(111)
        ax3.set_title("Air resistance/drag over time")
        ax3.set_xlabel("Time (s)")
        ax3.set_ylabel("Drag force (N)")
        ax3.plot(0, 0)
        ax3.grid(True)
        fig3.tight_layout()
        
        canvas3 = FigureCanvasTkAgg(fig3, master=root)
        canvas3.draw()
        canvas3.get_tk_widget().grid(row=0, column=7, rowspan=5, columnspan=2, sticky='nsew', padx=5, pady=5)
        
        # Create fourth figure (bottom-left)
        fig4 = Figure(figsize=(4, 2.5))
        fig4.patch.set_facecolor('grey')
        ax4 = fig4.add_subplot(111)
        ax4.set_title("Velocity in y against time")
        ax4.set_xlabel("Time (s)")
        ax4.set_ylabel("Velocity (m/s)")
        ax4.plot(0, 0)
        ax4.grid(True)
        fig4.tight_layout()
        
        canvas4 = FigureCanvasTkAgg(fig4, master=root)
        canvas4.draw()
        canvas4.get_tk_widget().grid(row=5, column=3, rowspan=5, columnspan=2, sticky='nsew', padx=5, pady=5)
        
        # Create fifth figure (bottom-middle)
        fig5 = Figure(figsize=(4, 2.5))
        fig5.patch.set_facecolor('grey')
        ax5 = fig5.add_subplot(111)
        ax5.set_title("Velocity in x against time")
        ax5.set_xlabel("Time (s)")
        ax5.set_ylabel("Velocity (m/s)")
        ax5.plot(0, 0)
        ax5.grid(True)
        fig5.tight_layout()
        
        canvas5 = FigureCanvasTkAgg(fig5, master=root)
        canvas5.draw()
        canvas5.get_tk_widget().grid(row=5, column=5, rowspan=5, columnspan=2, sticky='nsew', padx=5, pady=5)
        
        # Create sixth figure (bottom-right)
        fig6 = Figure(figsize=(4, 2.5))
        fig6.patch.set_facecolor('grey')
        ax6 = fig6.add_subplot(111)
        ax6.set_title("Velocity against time")
        ax6.set_xlabel("Time (s)")
        ax6.set_ylabel("Velocity (m/s)")
        ax6.plot(0, 0)
        ax6.grid(True)
        fig6.tight_layout()
        
        canvas6 = FigureCanvasTkAgg(fig6, master=root) 
        canvas6.draw()
        canvas6.get_tk_widget().grid(row=5, column=7, rowspan=5, columnspan=2, sticky='nsew', padx=5, pady=5)
        
        for i in range(3, 9):
            root.columnconfigure(i, weight=1)
        for i in range(10):
            root.rowconfigure(i, weight=1)
        
        # Setting up the labels for the variables on the screen 
        value_set_up(root)
        calculated_value_set_up(root)
        
        # Calculate button
        def calculate():
            # Get current values from labels
            v = my_function_v() or 0
            m = my_function_m() or 0
            a = my_function_a() or 0
            e = my_function_e() or 0
            c = my_function_c() or 0
            A = my_function_A() or 0
            p = my_function_p() or 0
            Cd = my_function_Cd() or 0
            E = my_function_E() or 0
            
            start_possible = Check_Values(a, m, e, c, p, v, A, Cd, E)
            if start_possible == True:
                calculation_iteration_subroutine(a, m, e, c, p, v, A, Cd, E,
                                                ax1, ax2, ax3, ax4, ax5, ax6,
                                                fig1, fig2, fig3, fig4, fig5, fig6, 
                                                canvas1, canvas2, canvas3, canvas4, canvas5, canvas6, root)
            else:
                messagebox.showerror("Missing Values", "Please enter all values before calculating")
        
        calculate_button = tk.Button(root, text="Calculate", command=calculate, bg='green', fg='white', font=('Arial', 14))
        calculate_button.grid(row=14, column=1, pady=10)
        
        root.mainloop()

    if __name__ == "__main__":
        main()

# system for casuilty calcualtion simulation 
def Casuilty_calculation():
    # setting constants
    ln = math.log
    pi = math.pi
    Na = 6.02*10**23
    C = 3.0*10**8
    C0 = 1.036


    #necessary subrotines

    # calulation for Activity (A)
    def activity(T,N,ln):
        A = ((ln(2)/T)*N)
        return A

    # calculation of intial number of nuclide
    def nuclide(m,M,Na):
        N = ((m/M)*Na)
        return N

    #  change in nuclide number
    def change_nuclide(ln,N,T,t):
        Change_N = ((ln(2)/T)*N*t)
        return Change_N

    # new number of nuclide
    def New_nuclide(N,Change_N):
        New_nuclide = N - Change_N
        N = New_nuclide
        return N

    #mass of final object after one half life using decay equation
    def final_mass(A,M,Na,T):
        mf = (A*M)/(Na*(ln(2)/T)) * math.exp(-(ln(2)/T)*T)
        return mf

    # finding change in mass
    def change_mass(m,mf):
        m_change = m - mf
        return m_change

    # setting new mass
    def new_mass(mf):
        m = mf
        return m

    #caculating energy from the explosion
    def energy_calculations(m_change,C):
        E = m_change*(C**2)
        return E

    #calculation of change of the radius over as extended time period
    def change_radius(C0,E,P,t):
        if t <= 0:
            return 0
        change_R = C0*(E/P)**(1/5)*t**(2/5)
        return change_R

    #caculating the new blast radius
    def calculating_new_blast(change_R,R):
        R_new = R + change_R
        return R_new

    #assigning new variable for R
    def reasigning_R(change_R,R):
        R = change_R + R
        return R

    # over pressure explosion
    def speed_shockwave(E,P_enviroment,t):
        if t <= 0:
            return 0
        U = (2/5)*((E/P_enviroment)**(1/5))*(t**-(3/5))
        return U

    # calculting the mach speed of the explosions
    def mach_speed(U):
        M = U/343
        return M

    # calculting pressure of explosion at a given time
    def pressure_calculation(M,P):
        delta_P = P*((2*1.4/1.4+1))
        return delta_P

    def Area_subroutine(r,pi):
        A = pi*(r)**2
        return A

    def population_subroutine(Pu,A):
        Pu_total = Pu*A
        return Pu_total

    # casuality estimate based on overpressure thresholds
    # over 100000 Pa near total casulities 95%
    # over 35000 Pa severe casulties 50%
    # over 17000 Pa moderate casulties 25%
    # below 17000 Pa minor casulties 5%
    def estimate_casulties(overpressure,population_in_blast):
        if overpressure > 100000:
            casulties = population_in_blast * 0.95
        elif overpressure > 35000:
            casulties = population_in_blast * 0.50
        elif overpressure > 17000:
            casulties = population_in_blast * 0.25
        else:
            casulties = population_in_blast * 0.05
        return casulties


    #defining nuclear countrols
    def nuclear_display(root):

        #error display panel
        def error_numeric():
            messagebox.showerror("Incorrect Error", "Error: please enter valid number")

        Entry_half_life = None
        Entry_mass = None
        Entry_molar_mass = None
        Entry__population_of_target = None
        Entry_maximum_radius_of_target = None
        Entry_pressure_of_enviroment = None

        title_label = None
        half_life_Label = None
        mass_Label = None
        molar_mass_label = None
        population_of_target_Label = None
        maximum_radius_of_target_label = None
        pressure_of_enviroment_label = None
        overpressure_explosion = None
        energy_explosion = None
        casulty_display = None

        # checks value is a compatible float
        def check_value_float(value):
            try:
                value = float(value)
                return True
            except ValueError:
                return False

        #change subroutine for the half life
        def change_T(T, half_life_Label):
            half_life_Label.config(text="Half life " + str(T) + " s")

        # change subroutine for the mass value
        def change_m(m, mass_Label):
            mass_Label.config(text="mass " + str(m) + " kg")

        # change subroutine for molar mass
        def change_M(M, molar_mass_label):
            molar_mass_label.config(text="Molar mass " + str(M) + " Kg")

        # change of population
        def change_P(Pu, population_of_target_Label):
            population_of_target_Label.config(text="population of target is " + str(Pu) + " per M^2")

        # change radius
        def change_r(r, maximum_radius_of_target_label):
            maximum_radius_of_target_label.config(text="radius of target " + str(r) + "m ")

        # change pressure
        def change_p(p, pressure_of_enviroment_label):
            pressure_of_enviroment_label.config(text="pressure of enviroment " + str(p) + " Pa")


        title_label = Label(root, text="Variables")
        half_life_Label = Label(root, text="Half life: not set")
        mass_Label = Label(root, text="mass: not set")
        molar_mass_label = Label(root, text="molar mass: not set")
        population_of_target_Label = Label(root, text="popouation of target per square meter is: not set")
        maximum_radius_of_target_label = Label(root, text=" radius of target: not set")
        pressure_of_enviroment_label = Label(root, text=" pressure of enviroment :not set")
        overpressure_explosion = Label(root, text="current overpressure of explosion is : not set")
        energy_explosion = Label(root, text="current energy output is : not set")
        casulty_display = Label(root, text="estimated casulties is : not set")

        title_label.grid(row=1, column=0)
        half_life_Label.grid(row=2, column=0)
        mass_Label.grid(row=3, column=0)
        molar_mass_label.grid(row=4, column=0)
        population_of_target_Label.grid(row=5, column=0)
        maximum_radius_of_target_label.grid(row=6, column=0)
        pressure_of_enviroment_label.grid(row=7, column=0)
        overpressure_explosion.grid(row=8, column=0)
        energy_explosion.grid(row=9, column=0)
        casulty_display.grid(row=10, column=0)

        Entry_half_life = tk.Entry(root)
        Entry_half_life.grid(row=2, column=1)
        Entry_mass = tk.Entry(root)
        Entry_mass.grid(row=3, column=1)
        Entry_molar_mass = tk.Entry(root)
        Entry_molar_mass.grid(row=4, column=1)
        Entry__population_of_target = tk.Entry(root)
        Entry__population_of_target.grid(row=5, column=1)
        Entry_maximum_radius_of_target = tk.Entry(root)
        Entry_maximum_radius_of_target.grid(row=6, column=1)
        Entry_pressure_of_enviroment = tk.Entry(root)
        Entry_pressure_of_enviroment.grid(row=7, column=1)

        # Entry for Half life
        def half_life_entered():
            result = my_function_T()
            if result == False:
                error_numeric()
            else:
                change_T(result, half_life_Label)

        velocity_button = tk.Button(root, text="Enter", command=half_life_entered)
        velocity_button.grid(row=2, column=2)

        # Entry for mass
        def mass_entered():
            result = my_function_m()
            if result == False:
                error_numeric()
            else:
                change_m(result, mass_Label)

        mass_button = tk.Button(root, text="Enter", command=mass_entered)
        mass_button.grid(row=3, column=2)

        # Entry for Molar mass
        def Molar_mass_entered():
            result = my_function_M()
            if result == False:
                error_numeric()
            else:
                change_M(result, molar_mass_label)

        angle_button = tk.Button(root, text="Enter", command=Molar_mass_entered)
        angle_button.grid(row=4, column=2)

        # Entry for population
        def population_target_entered():
            result = my_function_P()
            if result == False:
                error_numeric()
            else:
                change_P(result, population_of_target_Label)

        specific_heat_button = tk.Button(root, text="Enter", command=population_target_entered)
        specific_heat_button.grid(row=5, column=2)

        # Entry for radius of target
        def radius_entered():
            result = my_function_r()
            if result == False:
                error_numeric()
            else:
                change_r(result, maximum_radius_of_target_label)

        density_button = tk.Button(root, text="Enter", command=radius_entered)
        density_button.grid(row=6, column=2)

        # Entry for pressure of enviroment
        def enviroment_pressure_entered():
            result = my_function_p()
            if result == False:
                error_numeric()
            else:
                change_p(result, pressure_of_enviroment_label)

        drag_coeffecient_button = tk.Button(root, text="Enter", command=enviroment_pressure_entered)
        drag_coeffecient_button.grid(row=7, column=2)

        # collecting half life value
        def my_function_T():
            nonlocal Entry_half_life
            value = Entry_half_life.get()
            correct = check_value_float(value)
            if correct == True:
                v = value
                return float(v)
            else:
                return False

        # collecting mass value
        def my_function_m():
            nonlocal Entry_mass
            value = Entry_mass.get()
            correct = check_value_float(value)
            if correct == True:
                m = value
                return float(m)
            else:
                return False

        # collecting molar mass value
        def my_function_M():
            nonlocal Entry_molar_mass
            value = Entry_molar_mass.get()
            correct = check_value_float(value)
            if correct == True:
                M = value
                return float(M)
            else:
                return False

        # collecting population value
        def my_function_P():
            nonlocal Entry__population_of_target
            value = Entry__population_of_target.get()
            correct = check_value_float(value)
            if correct == True:
                Pu = value
                return float(Pu)
            else:
                return False

        # collecting radius value
        def my_function_r():
            nonlocal Entry_maximum_radius_of_target
            value = Entry_maximum_radius_of_target.get()
            correct = check_value_float(value)
            if correct == True:
                r = value
                return float(r)
            else:
                return False

        # collecting pressure value
        def my_function_p():
            nonlocal Entry_pressure_of_enviroment
            value = Entry_pressure_of_enviroment.get()
            correct = check_value_float(value)
            if correct == True:
                p = value
                return float(p)
            else:
                return False

        # setting up graphs
        # energy over time
        fig1 = Figure(figsize=(3, 2))
        fig1.patch.set_facecolor('grey')
        ax1 = fig1.add_subplot(111)
        ax1.set_title("Energy over time")
        ax1.set_xlabel("Time (s)")
        ax1.set_ylabel("Energy (J)")
        ax1.plot(0, 0)
        ax1.grid(True)
        fig1.tight_layout()

        canvas1 = FigureCanvasTkAgg(fig1, master=root)
        canvas1.draw()
        canvas1.get_tk_widget().grid(row=0, column=3, rowspan=5, columnspan=1, sticky='nsew', padx=2, pady=2)

        # blast radius over time
        fig2 = Figure(figsize=(3, 2))
        fig2.patch.set_facecolor('grey')
        ax2 = fig2.add_subplot(111)
        ax2.set_title("blast radius over time")
        ax2.set_xlabel("Time (s)")
        ax2.set_ylabel("Radius (m)")
        ax2.plot(0, 0)
        ax2.grid(True)
        fig2.tight_layout()

        canvas2 = FigureCanvasTkAgg(fig2, master=root)
        canvas2.draw()
        canvas2.get_tk_widget().grid(row=0, column=4, rowspan=5, columnspan=1, sticky='nsew', padx=2, pady=2)

        # overpressure over time
        fig3 = Figure(figsize=(3, 2))
        fig3.patch.set_facecolor('grey')
        ax3 = fig3.add_subplot(111)
        ax3.set_title("overpressure over time")
        ax3.set_xlabel("Time (s)")
        ax3.set_ylabel("Overpressure (Pa)")
        ax3.plot(0, 0)
        ax3.grid(True)
        fig3.tight_layout()

        canvas3 = FigureCanvasTkAgg(fig3, master=root)
        canvas3.draw()
        canvas3.get_tk_widget().grid(row=0, column=5, rowspan=5, columnspan=1, sticky='nsew', padx=2, pady=2)

        # casulties over time
        fig4 = Figure(figsize=(3, 2))
        fig4.patch.set_facecolor('grey')
        ax4 = fig4.add_subplot(111)
        ax4.set_title("casulties over time")
        ax4.set_xlabel("Time (s)")
        ax4.set_ylabel("Casulties")
        ax4.plot(0, 0)
        ax4.grid(True)
        fig4.tight_layout()

        canvas4 = FigureCanvasTkAgg(fig4, master=root)
        canvas4.draw()
        canvas4.get_tk_widget().grid(row=5, column=3, rowspan=5, columnspan=3, sticky='nsew', padx=2, pady=2)

        # Check values of entry before calculation
        def Check_Values(T, m, M, Pu, r, p):
            start = False
            total_correct = 0
            if T != 0:
                total_correct = total_correct + 1
            if m != 0:
                total_correct = total_correct + 1
            if M != 0:
                total_correct = total_correct + 1
            if Pu != 0:
                total_correct = total_correct + 1
            if r != 0:
                total_correct = total_correct + 1
            if p != 0:
                total_correct = total_correct + 1
            if total_correct == 6:
                start = True
            else:
                start = False
            return start

        # iteration subroutine
        def calculation_iteration_subroutine(T, m, M, Pu, r, p,
                                            ax1, ax2, ax3, ax4,
                                            fig1, fig2, fig3, fig4,
                                            canvas1, canvas2, canvas3, canvas4, root):

            # lists to store data for plotting
            time_values = []
            Energy_values = []
            overpressure_values = []
            casulty_values = []
            radius_values = []

            # calculate initial population of area based on provided values
            A = Area_subroutine(r, pi)
            Pu_total = population_subroutine(Pu, A)

            # calculation of intial energy
            N = nuclide(m, M, Na)
            A = activity(T, N, ln)
            mf = final_mass(A, M, Na, T)
            m_change = change_mass(m, mf)
            E_initial = energy_calculations(m_change, C)

            if E_initial <= 0:
                messagebox.showerror("Calculation Error", "Energy is zero or negative please check your inputs")
                return

            energy_explosion.config(text="current energy output is : " + str(E_initial) + " J")

            t = 0.01
            time_period = 0.1
            max_time = 60

            try:
                while t <= max_time:

                    # work out how much mass has decayed by this point in time
                    m_remaining = m * math.exp(-(ln(2)/T)*t)
                    m_change = m - m_remaining
                    E = E_initial * math.exp(-(t/max_time))

                    # calculate the radius of fireball from energy
                    R = change_radius(C0, E, p, t)

                    # shockwave speed and overpressure
                    U = speed_shockwave(E, p, t)
                    Mach = mach_speed(U)
                    # using mach number to get overpressure this should decrease over time
                    # making sure mach doesnt go below 1 otherwise you get negative pressure
                    if Mach < 1:
                        Mach = 1
                    delta_P = (2 * 1.4 * (Mach**2 - 1)) / (1.4 + 1) * p

                    # casulties based on overpressure
                    casulties = estimate_casulties(delta_P, Pu_total)

                    # add values to lists for graphing
                    time_values.append(t)
                    Energy_values.append(E)
                    radius_values.append(R)
                    overpressure_values.append(delta_P)
                    casulty_values.append(casulties)

                    # update graphs every 10 steps so it doesnt slow down too much
                    if len(time_values) % 10 == 0:
                        ax1.clear()
                        ax1.plot(time_values, Energy_values, 'b-')
                        ax1.set_title("Energy over time")
                        ax1.set_xlabel("Time (s)")
                        ax1.set_ylabel("Energy (J)")
                        ax1.grid(True)
                        fig1.tight_layout()
                        canvas1.draw()

                        ax2.clear()
                        ax2.plot(time_values, radius_values, 'r-')
                        ax2.set_title("blast radius over time")
                        ax2.set_xlabel("Time (s)")
                        ax2.set_ylabel("Radius (m)")
                        ax2.grid(True)
                        fig2.tight_layout()
                        canvas2.draw()

                        ax3.clear()
                        ax3.plot(time_values, overpressure_values, 'g-')
                        ax3.set_title("overpressure over time")
                        ax3.set_xlabel("Time (s)")
                        ax3.set_ylabel("Overpressure (Pa)")
                        ax3.grid(True)
                        fig3.tight_layout()
                        canvas3.draw()

                        # bar chart showing casulties deaths and survivors
                        deaths = casulty_values[-1] * 0.6
                        survivors = Pu_total - casulty_values[-1]
                        ax4.clear()
                        ax4.bar(["Casulties", "Deaths", "Survivors"], [casulty_values[-1], deaths, survivors], color=['orange', 'red', 'green'])
                        ax4.set_title("casulty breakdown")
                        ax4.set_ylabel("People")
                        ax4.grid(True)
                        fig4.tight_layout()
                        canvas4.draw()

                        # update the labels
                        casulty_display.config(text="estimated casulties is : " + str(int(casulty_values[-1])))
                        overpressure_explosion.config(text="current overpressure of explosion is : " + str(delta_P) + " Pa")
                        energy_explosion.config(text="current energy output is : " + str(E) + " J")

                        root.update()

                    t = t + time_period

                # update the casulty label at the end
                final_casulties = casulty_values[-1]
                casulty_display.config(text="estimated casulties is : " + str(int(final_casulties)))

                messagebox.showinfo("Simulation Complete",
                                    "Energy released: " + str(E_initial) + " J\n" +
                                    "Peak blast radius: " + str(radius_values[0]) + " m\n" +
                                    "Estimated casulties: " + str(int(final_casulties)))

            except Exception as ex:
                messagebox.showerror("Calculation Error", "An error occurred during calculation:\n" + str(ex))
                return

        # Calculate button
        def calculate():
            T = my_function_T() or 0
            m = my_function_m() or 0
            M = my_function_M() or 0
            Pu = my_function_P() or 0
            r = my_function_r() or 0
            p = my_function_p() or 0

            start_possible = Check_Values(T, m, M, Pu, r, p)
            if start_possible == True:
                calculation_iteration_subroutine(T, m, M, Pu, r, p,
                                                ax1, ax2, ax3, ax4,
                                                fig1, fig2, fig3, fig4,
                                                canvas1, canvas2, canvas3, canvas4, root)
            else:
                messagebox.showerror("Missing Values", "Please enter all values before calculating")

        calculate_button = tk.Button(root, text="Calculate", command=calculate, bg='green', fg='white', font=('Arial', 14))
        calculate_button.grid(row=14, column=1, pady=10)


    def main():
        root = tk.Tk()
        root.geometry("1600x900")
        root.state("zoomed")
        root.title("casulty calculation system")
        root.configure(bg='black')
        nuclear_display(root)
        root.update()
        root.mainloop()

    if __name__ == "__main__":
        main()

#reset password subroutine 
def reset_password():
    reset_screen = tk.Tk()
    reset_screen.geometry("600x300")
    reset_screen.title("reset password screen")
    reset_screen.configure(bg='black')
    
    # Create labels and entry fields
    Label(reset_screen, text="Please input the password you want to change to", 
          font=font1, bg='black', fg='grey').grid(row=0, column=0, columnspan=2, pady=10)
    
    entry_password1 = tk.Entry(reset_screen, width=30, show="*")
    entry_password1.grid(row=1, column=0, padx=5, pady=5)
    
    Label(reset_screen, text="Please confirm password below", 
          font=font1, bg='black', fg='grey').grid(row=2, column=0, columnspan=2, pady=10)
    
    entry_password2 = tk.Entry(reset_screen, show="*", width=30)
    entry_password2.grid(row=3, column=0, padx=5, pady=5)
    
    # Function to check and assign passwords
    def check_passwords():
        password1 = entry_password1.get()
        password2 = entry_password2.get()
        
        if password1 == password2 and password1 != "":
            global password
            password = password1
            Label(reset_screen, text="Password reset successful!", 
                  fg='green', bg='black').grid(row=5, column=0, columnspan=2)
            reset_screen.after(1500, reset_screen.destroy)
            encrypted_password = password_encryption(password)
            writing_Password_new(encrypted_password)
            return True
        else:
            Label(reset_screen, text="Passwords don't match or are empty!", 
                  fg='red', bg='black').grid(row=5, column=0, columnspan=2)
            return False
    
    # Single button to submit both passwords
    submit_button = Button(reset_screen, text="Submit", command=check_passwords, width=15)
    submit_button.grid(row=4, column=0, columnspan=2, pady=10)
    
    reset_screen.mainloop()


def login():
    global login_window, entry_label, login_button
    login_window = tk.Tk()
    login_window.geometry("600x300")  
    login_window.title("Projectile Motion Simulator - Login")
    login_window.configure(bg='black')

    #creating labels for screen 
    title_label = Label(login_window, text="Welcome", font=font2, bg='black', fg='white')
    title_label.grid(row=0, column=0, columnspan=3, pady=20)

    text_label = Label(login_window, text="Please login", font=font1, bg='black', fg='grey')
    text_label.grid(row=1, column=0, columnspan=3, pady=10)

    password_label = Label(login_window, text="Type password:", bg='black', fg='grey')
    password_label.grid(row=2, column=0, padx=10, pady=10, sticky='e')

    #creating the entry section for password and hiding input  
    entry_label = tk.Entry(login_window, show="*", width=30)
    entry_label.grid(row=2, column=1, pady=10)

    #button selection 
    login_button = Button(login_window, text="Login", command=verify_password, width=10)
    login_button.grid(row=3, column=1, pady=20)

    entry_label.bind('<Return>', lambda event: verify_password())

    login_window.mainloop()

login()
