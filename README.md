# Habbit-Tracker
import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Checkbox } from "@/components/ui/checkbox";
import { Button } from "@/components/ui/button";

const days = ["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"];

const defaultTasks = [
  "Wake up at 05:00","Study till 6:30","Gym","Study session2 3hrs","Study Session3 3hrs","Study Session4 3hrs","No fab","Post","12hrs Study"
];

export default function HabitTracker() {
  const [user, setUser] = useState(null);
  const [username, setUsername] = useState("");

  const [habits, setHabits] = useState([]);

  // Load user data
  useEffect(() => {
    const savedUser = localStorage.getItem("user");
    if (savedUser) {
      setUser(savedUser);
      const savedHabits = localStorage.getItem(`habits_${savedUser}`);
      if (savedHabits) {
        setHabits(JSON.parse(savedHabits));
      } else {
        setHabits(defaultTasks.map(name => ({ name, days: Array(7).fill(false) })));
      }
    }
  }, []);

  // Save habits
  useEffect(() => {
    if (user) {
      localStorage.setItem(`habits_${user}`, JSON.stringify(habits));
    }
  }, [habits, user]);

  const login = () => {
    if (!username) return;
    localStorage.setItem("user", username);
    setUser(username);

    const savedHabits = localStorage.getItem(`habits_${username}`);
    if (savedHabits) {
      setHabits(JSON.parse(savedHabits));
    } else {
      setHabits(defaultTasks.map(name => ({ name, days: Array(7).fill(false) })));
    }
  };

  const logout = () => {
    localStorage.removeItem("user");
    setUser(null);
  };

  const toggleHabit = (habitIndex, dayIndex) => {
    const updated = [...habits];
    updated[habitIndex].days[dayIndex] = !updated[habitIndex].days[dayIndex];
    setHabits(updated);
  };

  const updateHabitName = (index, value) => {
    const updated = [...habits];
    updated[index].name = value;
    setHabits(updated);
  };

  const dayProgress = (dayIndex) => {
    const done = habits.filter(h => h.days[dayIndex]).length;
    return Math.round((done / habits.length) * 100);
  };

  const overallProgress = () => {
    const total = habits.length * 7;
    const done = habits.reduce((acc, h) => acc + h.days.filter(Boolean).length, 0);
    return Math.round((done / total) * 100);
  };

  // LOGIN SCREEN
  if (!user) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-[#f4f7f2]">
        <Card className="w-80">
          <CardContent className="p-6 space-y-4">
            <h2 className="text-xl font-bold text-center">Login</h2>
            <Input
              placeholder="Enter username"
              value={username}
              onChange={(e) => setUsername(e.target.value)}
            />
            <Button onClick={login} className="w-full">Enter</Button>
          </CardContent>
        </Card>
      </div>
    );
  }

  return (
    <div className="p-6 bg-[#f4f7f2] min-h-screen">
      <div className="flex justify-between items-center mb-4">
        <h1 className="text-3xl font-bold">Habit Tracker</h1>
        <Button onClick={logout}>Logout</Button>
      </div>

      {/* Overall Progress */}
      <Card className="mb-6">
        <CardContent className="p-4">
          <h2 className="text-lg font-semibold mb-2">Overall Progress</h2>
          <div className="text-4xl font-bold text-green-600 text-center">
            {overallProgress()}%
          </div>
        </CardContent>
      </Card>

      {/* Habit Table */}
      <Card className="mb-6">
        <CardContent className="p-4 overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="text-left">
                <th>Habit</th>
                {days.map(d => (
                  <th key={d}>{d.slice(0,3)}</th>
                ))}
                <th>Progress</th>
              </tr>
            </thead>
            <tbody>
              {habits.map((habit, hIndex) => {
                const progress = Math.round(
                  (habit.days.filter(Boolean).length / 7) * 100
                );
                return (
                  <tr key={hIndex} className="border-t">
                    <td>
                      <Input
                        value={habit.name}
                        onChange={(e) => updateHabitName(hIndex, e.target.value)}
                      />
                    </td>
                    {habit.days.map((d, dayIndex) => (
                      <td key={dayIndex}>
                        <Checkbox
                          checked={d}
                          onCheckedChange={() => toggleHabit(hIndex, dayIndex)}
                        />
                      </td>
                    ))}
                    <td>
                      <div className="w-full bg-gray-200 h-2 rounded">
                        <div
                          className="bg-green-500 h-2 rounded"
                          style={{ width: `${progress}%` }}
                        />
                      </div>
                    </td>
                  </tr>
                );
              })}
            </tbody>
          </table>
        </CardContent>
      </Card>

      {/* Daily Cards */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        {days.map((day, dIndex) => (
          <Card key={day}>
            <CardContent className="p-4">
              <h3 className="font-semibold mb-2">{day}</h3>

              <div className="mb-2 text-green-600 font-bold">
                {dayProgress(dIndex)}%
              </div>

              <div className="space-y-1">
                {habits.map((h, hIndex) => (
                  <div key={hIndex} className="flex items-center gap-2">
                    <Checkbox
                      checked={h.days[dIndex]}
                      onCheckedChange={() => toggleHabit(hIndex, dIndex)}
                    />
                    <span className="text-sm">{h.name}</span>
                  </div>
                ))}
              </div>
            </CardContent>
          </Card>
        ))}
      </div>
    </div>
  );
}
