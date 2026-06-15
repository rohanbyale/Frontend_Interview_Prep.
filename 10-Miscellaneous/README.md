# 🎲 Miscellaneous — Git, Testing, Design Patterns, HR Questions

---

## GIT ESSENTIALS

### Common Git Commands
```bash
git init                    # start a new repo
git clone <url>             # copy a remote repo
git add .                   # stage all changes
git commit -m "message"     # save snapshot
git push origin main        # upload to remote
git pull                    # fetch + merge from remote

git branch feature/login    # create branch
git checkout feature/login  # switch to branch
git checkout -b feature/x   # create + switch in one step

git merge feature/login     # merge branch into current
git rebase main             # rebase current branch onto main

git stash                   # temporarily save uncommitted changes
git stash pop               # bring them back

git log --oneline           # see commit history
git diff                    # see unstaged changes
git reset --soft HEAD~1     # undo last commit (keep changes staged)
git reset --hard HEAD~1     # undo last commit (discard changes!)
```

---

## TESTING

### Unit vs Integration vs E2E
| Type | Tests | Speed | Tools |
|------|-------|-------|-------|
| Unit | Single function/component | Fast | Jest, Vitest |
| Integration | Multiple components together | Medium | React Testing Library |
| E2E | Full user flow in real browser | Slow | Playwright, Cypress |

```jsx
// Unit test with Jest + React Testing Library
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('increments count when button clicked', () => {
  render(<Counter />);
  
  const button = screen.getByRole('button', { name: /increment/i });
  const count = screen.getByText('0');
  
  fireEvent.click(button);
  
  expect(screen.getByText('1')).toBeInTheDocument();
});
```

---

## DESIGN PATTERNS

```javascript
// Singleton — one instance only
class ApiClient {
  static instance = null;
  static getInstance() {
    if (!this.instance) this.instance = new ApiClient();
    return this.instance;
  }
}

// Observer — subscribe to events
class EventEmitter {
  subscribers = {};
  on(event, fn) { (this.subscribers[event] ??= []).push(fn); }
  emit(event, data) { this.subscribers[event]?.forEach(fn => fn(data)); }
}

// Factory — create objects without specifying exact class
function createButton(type) {
  if (type === 'primary') return new PrimaryButton();
  if (type === 'danger') return new DangerButton();
}
```

---

## HR / BEHAVIORAL QUESTIONS

**"Tell me about yourself"**
> Keep it 2 minutes: Current role → Key skills → Why this company

**"Why do you want to join us?"**
> Research the company! Mention specific products, tech stack, or values.

**"Where do you see yourself in 5 years?"**
> Show growth mindset: "I want to grow into a senior/lead role, mentor others..."

**"Tell me about a challenging project"**
> Use STAR: Situation → Task → Action → Result

**"What's your biggest weakness?"**
> Be honest + show you're working on it: "I used to avoid giving critical feedback, but I've been practicing..."

**Salary negotiation:**
> Research market rates. Give a range, not a single number. "Based on my experience and market data, I'm looking at ₹X–₹Y."
