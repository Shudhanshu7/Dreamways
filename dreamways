/** @jsxImportSource https://esm.sh/react@18.2.0 */
import { createRoot } from "https://esm.sh/react-dom@18.2.0/client";
import React, { createContext, useContext, useEffect, useState } from "https://esm.sh/react@18.2.0";

// Firebase imports
import { initializeApp } from "https://esm.sh/firebase/app";
import {
  createUserWithEmailAndPassword,
  getAuth,
  onAuthStateChanged,
  signInWithEmailAndPassword,
  signOut,
} from "https://esm.sh/firebase/auth";
import {
  addDoc,
  collection,
  deleteDoc,
  doc,
  getDocs,
  getFirestore,
  query,
  where,
} from "https://esm.sh/firebase/firestore";

// Firebase configuration
const firebaseConfig = {
  apiKey: "AIzaSyATwrE4A-16-Tjqso7rx1qjwvmoJ_HiODE",
  authDomain: "dreamways-6d7cb.firebaseapp.com",
  projectId: "dreamways-6d7cb",
  storageBucket: "dreamways-6d7cb.firebasestorage.app",
  messagingSenderId: "53641792089",
  appId: "1:53641792089:web:6b32fb48ea3c9ef1b8bf48",
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

// Authentication Context
const AuthContext = createContext({
  user: null,
  login: () => {},
  logout: () => {},
  register: () => {},
});

// Saved Trips Context
const SavedTripsContext = createContext({
  savedTrips: [],
  saveTrip: async () => {},
  fetchSavedTrips: async () => {},
  deleteTrip: async () => {},
});

function SavedTripsProvider({ children }) {
  const [savedTrips, setSavedTrips] = useState([]);
  const { user } = useContext(AuthContext);

  const saveTrip = async (tripDetails) => {
    if (!user) {
      alert("Please log in to save trips");
      return false;
    }

    try {
      const tripRef = await addDoc(collection(db, "saved-trips"), {
        ...tripDetails,
        userId: user.uid,
        savedAt: new Date().toISOString(),
      });

      // Fetch updated trips after saving
      await fetchSavedTrips();

      alert("Trip saved successfully!");
      return true;
    } catch (error) {
      console.error("Error saving trip:", error);
      alert("Failed to save trip");
      return false;
    }
  };

  const fetchSavedTrips = async () => {
    if (!user) {
      setSavedTrips([]);
      return;
    }

    try {
      const q = query(
        collection(db, "saved-trips"),
        where("userId", "==", user.uid),
      );

      const querySnapshot = await getDocs(q);
      const trips = querySnapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data(),
      }));

      setSavedTrips(trips);
    } catch (error) {
      console.error("Error fetching saved trips:", error);
    }
  };

  const deleteTrip = async (tripId) => {
    if (!user) {
      alert("Please log in to delete trips");
      return false;
    }

    try {
      await deleteDoc(doc(db, "saved-trips", tripId));

      // Fetch updated trips after deleting
      await fetchSavedTrips();

      alert("Trip deleted successfully!");
      return true;
    } catch (error) {
      console.error("Error deleting trip:", error);
      alert("Failed to delete trip");
      return false;
    }
  };

  useEffect(() => {
    if (user) {
      fetchSavedTrips();
    }
  }, [user]);

  return (
    <SavedTripsContext.Provider value={{ savedTrips, saveTrip, fetchSavedTrips, deleteTrip }}>
      {children}
    </SavedTripsContext.Provider>
  );
}

function SavedTripsPage() {
  const { savedTrips, deleteTrip } = useContext(SavedTripsContext);
  const { user, logout } = useContext(AuthContext);

  if (!user) return null;

  return (
    <div className="saved-trips-container">
      <nav className="trip-planner-nav">
        <div className="nav-left">
          <img
            src="https://maxm-imggenurl.web.val.run/dreamways-logo-travel-app"
            alt="DreamWays Logo"
            className="logo"
          />
          <span className="brand-name">DreamWays</span>
        </div>
        <div className="nav-right">
          <span className="user-greeting">Welcome, {user.name}</span>
          <button onClick={logout} className="nav-btn logout-btn">
            Logout
          </button>
          <button onClick={() => window.location.href = "/"} className="back-btn">
            Home
          </button>
          <button onClick={() => window.location.href = "/trip-planner"} className="back-btn">
            Plan New Trip
          </button>
        </div>
      </nav>

      <div className="saved-trips-content">
        <h1>🌍 My Saved Trips</h1>
        {savedTrips.length === 0
          ? (
            <div className="no-trips-message">
              <p>You haven't saved any trips yet.</p>
              <button
                onClick={() => window.location.href = "/trip-planner"}
                className="plan-trip-btn"
              >
                Plan Your First Trip
              </button>
            </div>
          )
          : (
            <div className="saved-trips-list">
              {savedTrips.map((trip) => (
                <div key={trip.id} className="saved-trip-card">
                  <h2>Trip to {trip.destination}</h2>
                  <div className="trip-details">
                    <p>
                      <strong>Duration:</strong> {trip.days} days
                    </p>
                    <p>
                      <strong>Budget:</strong> {trip.budget}
                    </p>
                    <p>
                      <strong>Companions:</strong> {trip.companions}
                    </p>
                    <p>
                      <strong>Saved on:</strong> {new Date(trip.savedAt).toLocaleDateString()}
                    </p>
                  </div>
                  <details className="trip-plan-details">
                    <summary>View Detailed Itinerary</summary>
                    <pre>{trip.plan}</pre>
                  </details>
                  <div className="trip-card-actions">
                    <button
                      onClick={() => deleteTrip(trip.id)}
                      className="delete-trip-btn"
                    >
                      🗑️ Delete Trip
                    </button>
                  </div>
                </div>
              ))}
            </div>
          )}
      </div>
    </div>
  );
}

// Modify TripResults to include save trip functionality
function TripResults() {
  const [tripDetails, setTripDetails] = useState({
    destination: "",
    days: 0,
    budget: "",
    companions: "",
    plan: "",
  });
  const { user, logout } = useContext(AuthContext);
  const { saveTrip } = useContext(SavedTripsContext);

  useEffect(() => {
    const params = new URLSearchParams(window.location.search);
    setTripDetails({
      destination: params.get("destination") || "",
      days: parseInt(params.get("days") || "0"),
      budget: params.get("budget") || "",
      companions: params.get("companions") || "",
      plan: params.get("plan") || "",
    });
  }, []);

  const handleSaveTrip = async () => {
    const success = await saveTrip(tripDetails);
    if (success) {
      // Optionally redirect to saved trips page
      window.location.href = "/saved-trips";
    }
  };

  // If user is not logged in, don't render the results
  if (!user) return null;

  return (
    <div className="trip-results-container">
      <nav className="trip-planner-nav">
        <div className="nav-left">
          <img
            src="https://maxm-imggenurl.web.val.run/dreamways-logo-travel-app"
            alt="DreamWays Logo"
            className="logo"
          />
          <span className="brand-name">DreamWays</span>
        </div>
        <div className="nav-right">
          <span className="user-greeting">Welcome, {user.name}</span>
          <button onClick={logout} className="nav-btn logout-btn">
            Logout
          </button>
          <button onClick={() => window.location.href = "/"} className="back-btn">
            Home
          </button>
          <button onClick={() => window.location.href = "/trip-planner"} className="back-btn">
            Plan Another Trip
          </button>
        </div>
      </nav>
      <div className="trip-results-content">
        <h1>🌍 Your Personalized Trip to {tripDetails.destination}</h1>
        <div className="trip-summary">
          <div className="trip-details">
            <p>
              <strong>Duration:</strong> {tripDetails.days} days
            </p>
            <p>
              <strong>Budget:</strong> {tripDetails.budget}
            </p>
            <p>
              <strong>Travelling:</strong> {tripDetails.companions}
            </p>
          </div>
          <div className="trip-plan-scrollable">
            <h2>Detailed Itinerary</h2>
            <div className="trip-plan-scroll-container">
              <pre>{tripDetails.plan}</pre>
            </div>
          </div>
        </div>
        <div className="trip-actions">
          <button
            onClick={handleSaveTrip}
            className="save-trip-btn"
          >
            💾 Save This Trip
          </button>
        </div>
      </div>
    </div>
  );
}

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  // Check authentication on initial load
  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      if (currentUser) {
        setUser({
          name: currentUser.displayName || currentUser.email.split("@")[0],
          email: currentUser.email,
          uid: currentUser.uid,
        });
      } else {
        setUser(null);
      }
    });

    // Cleanup subscription on unmount
    return () => unsubscribe();
  }, []);

  const login = async (email, password) => {
    try {
      const userCredential = await signInWithEmailAndPassword(auth, email, password);
      alert("Login successful");
      window.location.href = "/trip-planner";
      return true;
    } catch (error) {
      console.error("Login failed", error);
      return false;
    }
  };

  const register = async (name, email, password) => {
    try {
      const userCredential = await createUserWithEmailAndPassword(auth, email, password);
      alert("Registration successful");
      window.location.href = "/trip-planner";
      return true;
    } catch (error) {
      console.error("Registration failed", error);
      return false;
    }
  };

  const logout = async () => {
    try {
      await signOut(auth);
      window.location.href = "/";
    } catch (error) {
      console.error("Logout failed", error);
    }
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, register }}>
      {children}
    </AuthContext.Provider>
  );
}

function LoginPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const { login } = useContext(AuthContext);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError("");
    const success = await login(email, password);
    if (!success) {
      setError("Invalid email or password");
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-form">
        <nav className="trip-planner-nav">
          <div className="nav-left">
            <img
              src="https://maxm-imggenurl.web.val.run/dreamways-logo-travel-app"
              alt="DreamWays Logo"
              className="logo"
            />
            <span className="brand-name">DreamWays</span>
          </div>
        </nav>
        <h2>Sign In</h2>
        {error && <p className="error-message">{error}</p>}
        <form onSubmit={handleSubmit}>
          <div>
            <label>Email</label>
            <input
              type="email"
              required
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              autoComplete="username"
            />
          </div>
          <div>
            <label>Password</label>
            <input
              type="password"
              required
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              autoComplete="current-password"
            />
          </div>
          <button type="submit" className="auth-btn">Sign In</button>
        </form>
        <p className="switch-auth-text">
          Don't have an account?{"  "}<a href="/register" className="switch-auth-link">Register</a>
        </p>
      </div>
    </div>
  );
}

function RegisterPage() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const { register } = useContext(AuthContext);

  // Password validation function
  const validatePassword = (pwd) => {
    const errors = [];

    // Check length
    if (pwd.length < 9) {
      errors.push("Password must be at least 9 characters long");
    }

    // Check for uppercase letter
    if (!/[A-Z]/.test(pwd)) {
      errors.push("Password must contain at least 1 uppercase letter");
    }

    // Check for lowercase letter
    if (!/[a-z]/.test(pwd)) {
      errors.push("Password must contain at least 1 lowercase letter");
    }

    // Check for number
    if (!/[0-9]/.test(pwd)) {
      errors.push("Password must contain at least 1 number");
    }

    // Check for special character
    if (!/[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(pwd)) {
      errors.push("Password must contain at least 1 special character");
    }

    return errors;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError("");

    // Validate password before registration
    const passwordErrors = validatePassword(password);

    if (passwordErrors.length > 0) {
      // Join errors into a single string for display
      setError(passwordErrors.join(", "));
      return;
    }

    const success = await register(name, email, password);
    if (!success) {
      setError("Registration failed. Email might already be in use.");
    }
  };

  return (
    <div className="auth-container">
      <div className="auth-form">
        <nav className="trip-planner-nav">
          <div className="nav-left">
            <img
              src="https://maxm-imggenurl.web.val.run/dreamways-logo-travel-app"
              alt="DreamWays Logo"
              className="logo"
            />
            <span className="brand-name">DreamWays</span>
          </div>
        </nav>
        <h2>Register</h2>
        {error && <p className="error-message">{error}</p>}
        <form onSubmit={handleSubmit}>
          <div>
            <label>Name</label>
            <input
              type="text"
              value={name}
              onChange={(e) => setName(e.target.value)}
              required
            />
          </div>
          <div>
            <label>Email</label>
            <input
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
            />
          </div>
          <div>
            <label>Password</label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required
            />
            <small className="password-requirements">
              Password must:
              <ul>
                <li>Be at least 9 characters long</li>
                <li>Contain at least 1 uppercase letter</li>
                <li>Contain at least 1 lowercase letter</li>
                <li>Contain at least 1 number</li>
                <li>Contain at least 1 special character</li>
              </ul>
            </small>
          </div>
          <button type="submit" className="auth-btn">Register</button>
        </form>
        <p className="switch-auth-text">
          Already have an account?{"  "}<a href="/login" className="switch-auth-link">Sign In</a>
        </p>
      </div>
    </div>
  );
}

function TripForm() {
  const [destination, setDestination] = useState("");
  const [days, setDays] = useState(3);
  const [budget, setBudget] = useState("medium");
  const [companions, setCompanions] = useState("single");
  const [isLoading, setIsLoading] = useState(false);
  const { user, logout } = useContext(AuthContext);

  // Redirect to login if not authenticated

  // useEffect(() => {
  //   if (!user) {
  //     window.location.href = "/login";
  //   }
  // }, [user]);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsLoading(true);

    try {
      const response = await fetch("/generate-trip", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ destination, days, budget, companions }),
      });

      if (!response.ok) throw new Error("Trip generation failed");

      const data = await response.json();
      window.location.href = `/trip-results?destination=${
        encodeURIComponent(destination)
      }&days=${days}&budget=${budget}&companions=${companions}&plan=${encodeURIComponent(data.plan)}`;
    } catch (error) {
      alert(`Error generating trip: ${error.message}`);
    } finally {
      setIsLoading(false);
    }
  };

  // If user is not logged in, don't render the form
  if (!user) return null;

  return (
    <div className="trip-planner">
      <nav className="trip-planner-nav">
        <div className="nav-left">
          <img
            src="https://maxm-imggenurl.web.val.run/dreamways-logo-travel-app"
            alt="DreamWays Logo"
            className="logo"
          />
          <span className="brand-name">DreamWays</span>
        </div>
        <div className="nav-right">
          <span className="user-greeting">Welcome, {user.name}</span>
          <button onClick={logout} className="nav-btn logout-btn">
            Logout
          </button>
          <button onClick={() => window.location.href = "/"} className="back-btn">
            Home
          </button>
          <button
            onClick={() => window.location.href = "/saved-trips"}
            className="nav-btn saved-trips-btn"
          >
            🌍 Saved Trips
          </button>
        </div>
      </nav>
      <h1>🌍 AI Trip Planner</h1>
      <form onSubmit={handleSubmit}>
        <div
          style={{
            height: "350px",
            backgroundImage:
              "url('https://media.worldnomads.com/create/learn/photography/landscape-photography-lead.jpg')",
            backgroundSize: "cover",
            backgroundPosition: "center",
            display: "flex",
            flexDirection: "column",
            justifyContent: "center",
            alignItems: "center",
            padding: "10px",
            borderRadius: "10px",
          }}
        >
          <label style={{ color: "white", fontWeight: "bold", marginBottom: "10px" }}>Destination:</label>
          <input
            type="text"
            value={destination}
            onChange={(e) => setDestination(e.target.value)}
            placeholder="Enter destination"
            required
            style={{
              height: "50px",
              width: "25%",
              padding: "10px",
              borderRadius: "5px",
              border: "1px solid #ccc",
              fontSize: "18px",
              fontWeight: "bold",
            }}
          />
        </div>

        <div style={{ display: "flex", height: "50px" }}>
          <label style={{ marginRight: "20px" }}>Number of Days:</label>
          <select
            value={days}
            onChange={(e) => setDays(Number(e.target.value))}
            style={{ marginRight: "20px", fontSize: "15px", fontWeight: "bold" }}
          >
            {[2, 3, 4, 5, 6, 7, 8, 9, 10].map(num => <option key={num} value={num}>{num}</option>)}
          </select>

          <label style={{ paddingTop: "13px", marginRight: "15px" }}>Budget:</label>
          <select
            value={budget}
            onChange={(e) => setBudget(e.target.value)}
            style={{ marginRight: "20px", fontSize: "15px", fontWeight: "bold" }}
          >
            <option value="low">💰 Low Budget</option>
            <option value="medium">💵 Medium Budget</option>
            <option value="expensive">💎 Expensive</option>
          </select>

          <label style={{ marginRight: "20px" }}>Travelling With:</label>
          <select
            value={companions}
            onChange={(e) => setCompanions(e.target.value)}
            style={{ fontSize: "15px", fontWeight: "bold" }}
          >
            <option value="single">🧑 Solo</option>
            <option value="couple">💑 Couple</option>
            <option value="family">👨‍👩‍👧‍👦 Family</option>
            <option value="friends">👥 Friends Group</option>
          </select>
        </div>
        <div style={{ display: "flex", justifyContent: "center", alignItems: "center" }}>
          <button
            type="submit"
            className="plan-trip-btn"
            disabled={isLoading}
            style={{ width: "190px", height: "57px" }}
          >
            {isLoading ? "Generating..." : "Plan My Trip"}
          </button>
        </div>
      </form>
    </div>
  );
}

function LandingPage() {
  const { user, logout } = useContext(AuthContext);

  return (
    <div className="landing-page">
      <nav className="trip-planner-nav">
        <div className="nav-left">
          <img
            src="https://maxm-imggenurl.web.val.run/dreamways-logo-travel-app"
            alt="DreamWays Logo"
            className="logo"
          />
          <span className="brand-name">DreamWays</span>
        </div>
        <div className="nav-right">
          {user
            ? (
              <>
                <span className="user-greeting">Welcome, {user.name}</span>
                <button
                  onClick={() => window.location.href = "/trip-planner"}
                  className="nav-btn"
                >
                  Plan a Trip
                </button>
                <button onClick={logout} className="nav-btn logout-btn">
                  Logout
                </button>
              </>
            )
            : (
              <>
                <button
                  onClick={() => window.location.href = "/login"}
                  className="nav-btn sign-in-btn"
                >
                  Sign In
                </button>
                <button
                  onClick={() => window.location.href = "/register"}
                  className="nav-btn register-btn"
                >
                  Register
                </button>
              </>
            )}
        </div>
      </nav>
      <div className="hero-section">
        <h1>✈️ Discover Your Next Adventure with AI: Personalized Itineraries at Your Fingertips.</h1>
        <p>
          Your personal trip planner and travel curator, creating custom itineraries tailored to your interests and
          budget.
        </p>
        <div className="cta-buttons">
          {user
            ? (
              <button
                onClick={() => window.location.href = "/trip-planner"}
                className="cta-button get-started-btn"
              >
                Plan Your Trip 🌍
              </button>
            )
            : (
              <button
                onClick={() => window.location.href = "/register"}
                className="cta-button get-started-btn"
              >
                Get Started 🌍
              </button>
            )}
        </div>
      </div>
      {/* Existing features section remains the same */}
      <div className="features-section">
        <div
          className="feature"
          style={{
            border: "2px solid #ddd", // Light gray border
            boxShadow: "4px 4px 15px rgba(0, 0, 0, 0.1)", // Soft shadow for depth
            borderRadius: "10px", // Rounded corners for a modern look
            padding: "20px", // Adds spacing inside the box
            textAlign: "center", // Centers content
            backgroundColor: "#fff", // White background for contrast
            width: "250px", // Adjust width as needed
          }}
        >
          <span style={{ fontSize: "30px" }}>🤖</span>
          <h3 style={{ margin: "10px 0" }}>AI-Powered Planning</h3>
          <p style={{ color: "#666" }}>Personalized itineraries tailored to your preferences</p>
        </div>

        <div
          className="feature"
          style={{
            border: "2px solid #ddd", // Light gray border
            boxShadow: "4px 4px 15px rgba(0, 0, 0, 0.1)", // Soft shadow for depth
            borderRadius: "10px", // Rounded corners for a modern look
            padding: "20px", // Adds spacing inside the box
            textAlign: "center", // Centers content
            backgroundColor: "#fff", // White background for contrast
            width: "250px", // Adjust width as needed
          }}
        >
          <span>💰</span>
          <h3>Budget Friendly</h3>
          <p>Options for every budget, from economical to luxurious</p>
        </div>
        <div
          className="feature"
          style={{
            border: "2px solid #ddd", // Light gray border
            boxShadow: "4px 4px 15px rgba(0, 0, 0, 0.1)", // Soft shadow for depth
            borderRadius: "10px", // Rounded corners for a modern look
            padding: "20px", // Adds spacing inside the box
            textAlign: "center", // Centers content
            backgroundColor: "#fff", // White background for contrast
            width: "250px", // Adjust width as needed
          }}
        >
          <span>🔄</span>
          <h3>Flexible Options</h3>
          <p>Plan for solo, couples, families, and friend groups</p>
        </div>
      </div>
    </div>
  );
}

// Modify client function to include SavedTripsProvider and new route
function client() {
  const path = window.location.pathname;
  const root = createRoot(document.getElementById("root"));

  root.render(
    <AuthProvider>
      <SavedTripsProvider>
        {path === "/" && <LandingPage />}
        {path === "/login" && <LoginPage />}
        {path === "/register" && <RegisterPage />}
        {path === "/trip-planner" && <TripForm />}
        {path === "/trip-results" && <TripResults />}
        {path === "/saved-trips" && <SavedTripsPage />}
      </SavedTripsProvider>
    </AuthProvider>,
  );
}

if (typeof document !== "undefined") { client(); }

export default async function server(request: Request): Promise<Response> {
  const url = new URL(request.url);

  if (request.method === "POST" && url.pathname === "/generate-trip") {
    const { destination, days, budget, companions } = await request.json();

    const { OpenAI } = await import("https://esm.town/v/std/openai");
    const openai = new OpenAI();

    const prompt =
      `Create a detailed ${days}-day travel itinerary for ${destination} for a ${companions} traveler with a ${budget} budget. Include recommended activities, estimated costs, transportation, and dining suggestions.`;

    const completion = await openai.chat.completions.create({
      messages: [{ role: "user", content: prompt }],
      model: "gpt-4o-mini",
      max_tokens: 5000,
    });

    const plan = completion.choices[0].message.content;

    return new Response(JSON.stringify({ plan }), {
      headers: { "Content-Type": "application/json" },
    });
  }

  return new Response(
    `
    <html>
      <head>
        <title>DreamWays - AI Trip Planner</title>
        <style>${css}</style>
        <link rel="icon" href="https://maxm-imggenurl.web.val.run/dreamways-logo-travel-app" type="image/png" />
      </head>
      <body>
        <div id="root"></div>
        <script src="https://esm.town/v/std/catch"></script>
        <script type="module" src="${import.meta.url}"></script>
        <a href="${
      import.meta.url.replace("esm.town", "val.town")
    }" target="_top" style="position: fixed; bottom: 10px; right: 10px; color: #666; text-decoration: none;">View Source</a>
      </body>
    </html>
  `,
    {
      headers: { "Content-Type": "text/html" },
    },
  );
}

// Add new CSS for saved trips (append to existing CSS)
let css = `
  /* Existing CSS remains the same, adding these new styles: */
  .saved-trips-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
  }

  .saved-trips-content {
    background-color: white;
    padding: 30px;
    border-radius: 10px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  }

  .saved-trips-list {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 20px;
  }

  .saved-trip-card {
    background-color: #f9f9f9;
    border-radius: 8px;
    padding: 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  .saved-trip-card h2 {
    margin-top: 0;
    color: #333;
  }

  .trip-plan-details {
    margin-top: 15px;
  }

  .trip-plan-details summary {
    cursor: pointer;
    font-weight: bold;
    color: #666;
  }

  .trip-plan-details pre {
    background-color: #f4f4f4;
    padding: 15px;
    border-radius: 6px;
    max-height: 300px;
    overflow-y: auto;
  }

  .no-trips-message {
    text-align: center;
    padding: 50px 20px;
    background-color: #f9f9f9;
    border-radius: 10px;
  }

  .trip-card-actions {
    margin-top: 15px;
    text-align: center;
  }

  .delete-trip-btn {
    background-color: #ff4d4d;
    color: white;
    border: none;
    padding: 10px 15px;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .delete-trip-btn:hover {
    background-color: #ff3333;
  }

  .trip-actions {
    margin-top: 20px;
    text-align: center;
  }

  .save-trip-btn {
    padding: 12px 24px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.3s ease, transform 0.2s ease;
  }

  .save-trip-btn:hover {
    background-color: #45a049;
    transform: scale(1.05);
  }
  .auth-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f4f4f4;
}

.auth-form {
  background-color: white;
  padding: 40px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  width: 400px;
  max-width: 90%;
}

.auth-form h2 {
  text-align: center;
  margin-bottom: 20px;
}

.auth-form form div {
  margin-bottom: 15px;
}

.auth-form label {
  display: block;
  margin-bottom: 5px;
}

.auth-form input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.auth-btn {
  width: 100%;
  padding: 10px;
  background-color: black;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.auth-btn:hover {
  background-color: #333;
}

.switch-auth-text {
  text-align: center;
  margin-top: 15px;
}

.switch-auth-link {
  color: black;
  text-decoration: underline;
}

.error-message {
  color: red;
  text-align: center;
  margin-bottom: 15px;
}

.user-greeting {
  margin-right: 15px;
  font-weight: bold;
}

.logout-btn {
  background-color: #ff4d4d;
}

.register-btn {
  background-color: #4CAF50;
}

body {
  font-family: 'Arial', sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f4f4f4;
  line-height: 1.6;
}

.trip-planner-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 30px;
  background-color: #FF8C00;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.nav-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.nav-right {
  display: flex;
  gap: 15px;
}

.nav-btn {
  background-color: black;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s ease;
}

.nav-btn:hover {
  background-color: #333;
  transform: scale(1.05);
}

.logo {
  width: 50px;
  height: 50px;
  border-radius: 50%;
}

.brand-name {
  font-size: 1.5em;
  font-weight: bold;
  color: #333;
}

.back-btn {
  background-color: black;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s ease;;
}
.back-btn:hover {
  background-color: #333;
  transform: scale(1.05);
}

.trip-planner {
  max-width: 100%;
  margin: 0 auto;
  padding: 20px;
}

.landing-page {
  text-align: center;
  padding: 20px;
}

.hero-section {
  background-color: #f9f9f9;
  padding: 50px 20px;
}

.hero-section h1 {
  font-size: 2.5em;
  margin-bottom: 20px;
}

.cta-buttons {
  margin-top: 30px;
}

.get-started-btn {
  padding: 12px 24px;
  font-size: 18px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  background-color: black;
  color: white;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.get-started-btn:hover {
  background-color: #424242;
  transform: scale(1.05);
}

.features-section {
  display: flex;
  justify-content: space-around;
  padding: 50px 20px;
  background-color: white;
}

.feature {
  max-width: 300px;
  padding: 20px;
}

.feature span {
  font-size: 3em;
}

form {
  background-color: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  width: 100%;
  box-sizing: border-box;

}

form div {
  margin-bottom: 15px;
}

input, select {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.plan-trip-btn {
  padding: 8px 16px;
  background-color: black;
  color: #FAF5FF;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  border: none;
  font-size: 16px;
  width: 100%;
}

.plan-trip-btn:hover {
  background-color: #1F2937;
  transform: scale(1.02);
}

.plan-trip-btn:disabled {
  background-color: #666;
  cursor: not-allowed;
  transform: none;
}

.trip-results-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.trip-results-content {
  background-color: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  margin-top: 20px;
}

.trip-summary {
  display: flex;
  gap: 30px;
}

.trip-details {
  flex: 1;
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
}

.trip-plan {
  flex: 2;
}

.trip-plan pre {
  background-color: #f4f4f4;
  padding: 20px;
  border-radius: 8px;
  white-space: pre-wrap;
  word-wrap: break-word;
  font-family: 'Arial', sans-serif;
  line-height: 1.6;
}

.trip-plan-scrollable {
  flex: 2;
  display: flex;
  flex-direction: column;
}

.trip-plan-scroll-container {
  flex-grow: 1;
  overflow-y: auto;
  max-height: 500px; /* Adjust as needed */
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 15px;
  background-color: #f9f9f9;
}

.trip-plan-scroll-container pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  margin: 0;
  font-family: 'Arial', sans-serif;
  line-height: 1.6;
  font-size: 0.9em;
}

/* Scrollbar styling for webkit browsers */
.trip-plan-scroll-container::-webkit-scrollbar {
  width: 10px;
}

.trip-plan-scroll-container::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

.trip-plan-scroll-container::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
}

.trip-plan-scroll-container::-webkit-scrollbar-thumb:hover {
  background: #555;
}
`;
