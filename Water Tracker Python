import streamlit as st
import json
import os
from datetime import date

# ── Config ────────────────────────────────────────────────────────────────────
DATA_FILE = "water_tracker_data.json"
GOAL_ML   = 3000

# ── Storage helpers ───────────────────────────────────────────────────────────

def load_state() -> dict:
    """Load persisted state from JSON file. Returns default state if missing."""
    if os.path.exists(DATA_FILE):
        try:
            with open(DATA_FILE, "r") as f:
                return json.load(f)
        except (json.JSONDecodeError, KeyError):
            pass
    return {"date": str(date.today()), "total": 0}


def save_state(total: int) -> None:
    """Persist current date and total to JSON file."""
    with open(DATA_FILE, "w") as f:
        json.dump({"date": str(date.today()), "total": total}, f)


# ── Business logic ────────────────────────────────────────────────────────────

def reset_if_new_day(state: dict) -> dict:
    """Reset total to 0 if the stored date is not today."""
    if state.get("date") != str(date.today()):
        state = {"date": str(date.today()), "total": 0}
        save_state(0)
    return state


def add_water(amount: int) -> None:
    """Increment today's total by `amount` ml and persist."""
    st.session_state.today_total_ml += amount
    save_state(st.session_state.today_total_ml)


def calculate_metrics(total: int, goal: int) -> tuple[float, int]:
    """Return (completion_pct capped at 100, remaining_ml floored at 0)."""
    completion = min((total / goal) * 100, 100.0)
    remaining  = max(goal - total, 0)
    return completion, remaining


# ── Bootstrap session state ───────────────────────────────────────────────────

def init_state() -> None:
    if "today_total_ml" not in st.session_state:
        state = load_state()
        state = reset_if_new_day(state)
        st.session_state.today_total_ml = state["total"]


# ── UI ────────────────────────────────────────────────────────────────────────

def render_ui() -> None:
    # ── Page config ──────────────────────────────────────────────────────────
    st.set_page_config(
        page_title="Water Tracker",
        page_icon="💧",
        layout="centered",
    )

    # ── Custom CSS ───────────────────────────────────────────────────────────
    st.markdown("""
    <style>
    @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:wght@300;400;600&display=swap');

    html, body, [class*="css"] {
        font-family: 'DM Sans', sans-serif;
    }

    /* ── Background ── */
    .stApp {
        background: linear-gradient(160deg, #0d1b2a 0%, #112236 60%, #0a2a3a 100%);
        color: #e0f0ff;
    }

    /* ── Hide Streamlit chrome ── */
    #MainMenu, footer, header { visibility: hidden; }

    /* ── Title ── */
    .tracker-title {
        font-family: 'Space Mono', monospace;
        font-size: 2.2rem;
        font-weight: 700;
        color: #7dd3fc;
        letter-spacing: -1px;
        margin-bottom: 0.2rem;
    }
    .tracker-date {
        font-size: 0.82rem;
        color: #4a7a99;
        font-family: 'Space Mono', monospace;
        margin-bottom: 2rem;
    }

    /* ── Metric cards ── */
    .metric-row {
        display: flex;
        gap: 1rem;
        margin-bottom: 1.6rem;
    }
    .metric-card {
        flex: 1;
        background: rgba(255,255,255,0.04);
        border: 1px solid rgba(125,211,252,0.12);
        border-radius: 14px;
        padding: 1.1rem 1rem 0.9rem;
        text-align: center;
    }
    .metric-label {
        font-size: 0.7rem;
        text-transform: uppercase;
        letter-spacing: 1.5px;
        color: #4a7a99;
        margin-bottom: 0.35rem;
    }
    .metric-value {
        font-family: 'Space Mono', monospace;
        font-size: 1.55rem;
        font-weight: 700;
        color: #e0f0ff;
    }
    .metric-value.highlight {
        color: #7dd3fc;
        font-size: 1.9rem;
    }
    .metric-unit {
        font-size: 0.75rem;
        color: #4a7a99;
        margin-top: 0.15rem;
    }

    /* ── Progress bar container ── */
    .progress-wrap {
        background: rgba(255,255,255,0.06);
        border-radius: 999px;
        height: 12px;
        overflow: hidden;
        margin-bottom: 0.45rem;
    }
    .progress-fill {
        height: 100%;
        border-radius: 999px;
        background: linear-gradient(90deg, #38bdf8, #7dd3fc);
        transition: width 0.4s ease;
    }
    .progress-label {
        font-size: 0.72rem;
        color: #4a7a99;
        text-align: right;
        font-family: 'Space Mono', monospace;
        margin-bottom: 2rem;
    }

    /* ── Buttons ── */
    .stButton > button {
        width: 100%;
        background: rgba(56, 189, 248, 0.1);
        border: 1px solid rgba(56, 189, 248, 0.3);
        color: #7dd3fc;
        border-radius: 10px;
        padding: 0.7rem 0;
        font-family: 'Space Mono', monospace;
        font-size: 1rem;
        font-weight: 700;
        letter-spacing: 0.5px;
        transition: background 0.2s, border-color 0.2s, transform 0.1s;
        cursor: pointer;
    }
    .stButton > button:hover {
        background: rgba(56, 189, 248, 0.2);
        border-color: rgba(56, 189, 248, 0.6);
        transform: translateY(-1px);
    }
    .stButton > button:active {
        transform: translateY(0px);
    }

    /* ── Goal reached banner ── */
    .goal-banner {
        background: linear-gradient(90deg, rgba(56,189,248,0.15), rgba(125,211,252,0.08));
        border: 1px solid rgba(56,189,248,0.35);
        border-radius: 12px;
        padding: 0.9rem 1.2rem;
        text-align: center;
        font-size: 0.95rem;
        color: #7dd3fc;
        font-family: 'Space Mono', monospace;
        margin-top: 1.4rem;
    }

    /* ── Section label ── */
    .section-label {
        font-size: 0.7rem;
        text-transform: uppercase;
        letter-spacing: 2px;
        color: #4a7a99;
        margin-bottom: 0.8rem;
    }
    </style>
    """, unsafe_allow_html=True)

    # ── Header ───────────────────────────────────────────────────────────────
    st.markdown('<div class="tracker-title">💧 Water Tracker</div>', unsafe_allow_html=True)
    st.markdown(f'<div class="tracker-date">{date.today().strftime("%A, %d %B %Y")}</div>', unsafe_allow_html=True)

    # ── Metrics ──────────────────────────────────────────────────────────────
    total      = st.session_state.today_total_ml
    completion, remaining = calculate_metrics(total, GOAL_ML)

    st.markdown(f"""
    <div class="metric-row">
        <div class="metric-card">
            <div class="metric-label">Goal</div>
            <div class="metric-value">{GOAL_ML:,}</div>
            <div class="metric-unit">ml</div>
        </div>
        <div class="metric-card">
            <div class="metric-label">Today</div>
            <div class="metric-value">{total:,}</div>
            <div class="metric-unit">ml</div>
        </div>
        <div class="metric-card">
            <div class="metric-label">Completion</div>
            <div class="metric-value highlight">{completion:.0f}%</div>
            <div class="metric-unit">&nbsp;</div>
        </div>
        <div class="metric-card">
            <div class="metric-label">Remaining</div>
            <div class="metric-value">{remaining:,}</div>
            <div class="metric-unit">ml</div>
        </div>
    </div>
    """, unsafe_allow_html=True)

    # ── Progress bar ─────────────────────────────────────────────────────────
    st.markdown(f"""
    <div class="progress-wrap">
        <div class="progress-fill" style="width:{completion}%"></div>
    </div>
    <div class="progress-label">{completion:.1f}% of daily goal</div>
    """, unsafe_allow_html=True)

    # ── Add water buttons ────────────────────────────────────────────────────
    st.markdown('<div class="section-label">Add Water</div>', unsafe_allow_html=True)

    col1, col2, col3 = st.columns(3)
    with col1:
        if st.button("+ 250 ml"):
            add_water(250)
            st.rerun()
    with col2:
        if st.button("+ 500 ml"):
            add_water(500)
            st.rerun()
    with col3:
        if st.button("+ 1000 ml"):
            add_water(1000)
            st.rerun()

    # ── Goal reached banner ──────────────────────────────────────────────────
    if total >= GOAL_ML:
        st.markdown(
            '<div class="goal-banner">🎉 Daily goal reached! Great job staying hydrated.</div>',
            unsafe_allow_html=True,
        )


# ── Entry point ───────────────────────────────────────────────────────────────

def main() -> None:
    init_state()
    render_ui()


if __name__ == "__main__":
    main()
