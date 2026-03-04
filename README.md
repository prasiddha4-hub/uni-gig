import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

export default function UniGigApp() {
  const MIN_WAGE = 12.21;
  const MAX_WEEKLY_HOURS = 20;

  const [user, setUser] = useState(null);
  const [uniId, setUniId] = useState("");
  const [studentType, setStudentType] = useState("home");
  const [shareCode, setShareCode] = useState("");
  const [hoursWorked, setHoursWorked] = useState(0);
  const [isClockedIn, setIsClockedIn] = useState(false);
  const [startTime, setStartTime] = useState(null);
  const [locationStatus, setLocationStatus] = useState("Not verified");

  const earnings = (hoursWorked * MIN_WAGE).toFixed(2);

  const handleLogin = () => {
    if (!uniId) return alert("Enter valid University ID");
    if (studentType === "international" && !shareCode)
      return alert("International students must provide share code");

    setUser({ uniId, studentType });
  };

  const verifyGPS = () => {
    if (!navigator.geolocation) {
      setLocationStatus("GPS not supported");
      return;
    }

    navigator.geolocation.getCurrentPosition(
      () => setLocationStatus("Location Verified ✅"),
      () => setLocationStatus("Location Denied ❌")
    );
  };

  const clockIn = () => {
    if (hoursWorked >= MAX_WEEKLY_HOURS && user.studentType === "international") {
      alert("You have reached 20 hours limit this week.");
      return;
    }

    if (locationStatus !== "Location Verified ✅") {
      alert("Please verify GPS before clocking in.");
      return;
    }

    setStartTime(Date.now());
    setIsClockedIn(true);
  };

  const clockOut = () => {
    const worked = (Date.now() - startTime) / (1000 * 60 * 60);
    const totalHours = hoursWorked + worked;

    if (user.studentType === "international" && totalHours > MAX_WEEKLY_HOURS) {
      setHoursWorked(MAX_WEEKLY_HOURS);
    } else {
      setHoursWorked(totalHours);
    }

    setIsClockedIn(false);
  };

  if (!user) {
    return (
      <div className="flex items-center justify-center min-h-screen bg-gray-100 p-4">
        <Card className="w-full max-w-md p-6 shadow-xl rounded-2xl">
          <CardContent className="space-y-4">
            <h1 className="text-2xl font-bold text-center">Uni-Gig Login</h1>
            <Input
              placeholder="University ID"
              value={uniId}
              onChange={(e) => setUniId(e.target.value)}
            />
            <select
              className="w-full p-2 border rounded-lg"
              onChange={(e) => setStudentType(e.target.value)}
            >
              <option value="home">Home Student</option>
              <option value="international">International Student</option>
            </select>
            {studentType === "international" && (
              <Input
                placeholder="Enter Share Code"
                value={shareCode}
                onChange={(e) => setShareCode(e.target.value)}
              />
            )}
            <Button className="w-full" onClick={handleLogin}>
              Login with Uni ID
            </Button>
          </CardContent>
        </Card>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-50 p-6 grid gap-6">
      <h1 className="text-3xl font-bold">Welcome to Uni-Gig</h1>

      <Card className="p-4 rounded-2xl shadow-md">
        <CardContent className="space-y-2">
          <p><strong>Student Type:</strong> {user.studentType}</p>
          <p><strong>Hours Worked:</strong> {hoursWorked.toFixed(2)} / {user.studentType === "international" ? 20 : "Unlimited"}</p>
          <p><strong>Total Earnings:</strong> £{earnings} GBP</p>
          <p><strong>Location Status:</strong> {locationStatus}</p>
        </CardContent>
      </Card>

      <Card className="p-4 rounded-2xl shadow-md">
        <CardContent className="space-y-4">
          <h2 className="text-xl font-semibold">Shift Control</h2>
          <Button onClick={verifyGPS}>Verify GPS</Button>
          {!isClockedIn ? (
            <Button onClick={clockIn} disabled={user.studentType === "international" && hoursWorked >= MAX_WEEKLY_HOURS}>
              Clock In
            </Button>
          ) : (
            <Button onClick={clockOut}>Clock Out</Button>
          )}
        </CardContent>
      </Card>

      <Card className="p-4 rounded-2xl shadow-md">
        <CardContent className="space-y-2">
          <h2 className="text-xl font-semibold">Available Opportunities</h2>
          {(user.studentType === "international" && hoursWorked >= MAX_WEEKLY_HOURS) ? (
            <p className="text-red-500">You have reached your 20-hour weekly limit. Applications locked.</p>
          ) : (
            <ul className="list-disc pl-5">
              <li>Part-Time Retail Assistant</li>
              <li>Warehouse Operative</li>
              <li>Campus Ambassador Internship</li>
              <li>Marketing Intern (Part-Time)</li>
            </ul>
          )}
        </CardContent>
      </Card>

      <Card className="p-4 rounded-2xl shadow-md">
        <CardContent className="space-y-2">
          <h2 className="text-xl font-semibold">Rules & Training</h2>
          <ul className="list-disc pl-5">
            <li>UK Work Regulation (20 hrs/week for International Students)</li>
            <li>National Minimum Wage: £12.21 GBP</li>
            <li>Health & Safety Training Video</li>
            <li>Right to Work & Share Code Verification</li>
          </ul>
        </CardContent>
      </Card>
    </div>
  );
}
