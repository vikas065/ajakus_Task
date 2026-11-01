#  Part 2 test run before fixes

// @vitest-environment node
// Part 2 starter — do not modify the test descriptions or setup
// Requires the dev server running on http://localhost:3000

import { describe, it, expect, beforeAll } from "vitest";

const BASE_URL = process.env.TEST_BASE_URL || "http://localhost:3000";

async function login(email: string): Promise<string> {
  const res = await fetch(`${BASE_URL}/api/auth/login`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password: "password123" }),
  });
  const data = (await res.json()) as { token: string };
  return data.token;
}

let tokens: { meera: string; arjun: string; dev: string };
let projectId: string;
let taskId: string;

beforeAll(async () => {
  const [meera, arjun, dev] = await Promise.all([
    login("meera@taskboard.dev"),
    login("arjun@taskboard.dev"),
    login("dev@example.com"),
  ]);
  tokens = { meera, arjun, dev };

  const projectsRes = await fetch(`${BASE_URL}/api/projects`, {
    headers: { Authorization: `Bearer ${meera}` },
  });
  const { projects } = (await projectsRes.json()) as {
    projects: { id: string; name: string }[];
  };
  projectId = projects.find((p) => p.name === "Q3 Launch")!.id;

  const tasksRes = await fetch(`${BASE_URL}/api/projects/${projectId}/tasks`, {
    headers: { Authorization: `Bearer ${meera}` },
  });
  const { tasks } = (await tasksRes.json()) as { tasks: { id: string }[] };
  taskId = tasks[0].id;
});

describe("task access control", () => {
  // Test A
  it("a viewer cannot update a task", async () => {
    const res = await fetch(`${BASE_URL}/api/tasks/${taskId}`, {
      method: "PATCH",
      headers: {
        Authorization: `Bearer ${tokens.dev}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title: "viewer update attempt" }),
    });
    expect(res.status).toBe(403);
  });

  // Test B
  it("a viewer cannot create a task", async () => {
    const res = await fetch(`${BASE_URL}/api/projects/${projectId}/tasks`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${tokens.dev}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title: "viewer create attempt" }),
    });
    expect(res.status).toBe(401);
  });

  // Test C
  it("a member can create a task", async () => {
    const res = await fetch(`${BASE_URL}/api/projects/${projectId}/tasks`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${tokens.arjun}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title: "member create — baseline" }),
    });
    expect(res.status).toBe(201);
  });
});

#  Part 2 test run after fixes


// @vitest-environment node
// Part 2 starter — do not modify the test descriptions or setup
// Requires the dev server running on http://localhost:3000

import { describe, it, expect, beforeAll } from "vitest";

const BASE_URL = process.env.TEST_BASE_URL || "http://localhost:3000";

async function login(email: string): Promise<string> {
  const res = await fetch(`${BASE_URL}/api/auth/login`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password: "password123" }),
  });
  const data = (await res.json()) as { token: string };
  return data.token;
}

let tokens: { meera: string; arjun: string; dev: string };
let projectId: string;
let taskId: string;

beforeAll(async () => {
  const [meera, arjun, dev] = await Promise.all([
    login("meera@taskboard.dev"),
    login("arjun@taskboard.dev"),
    login("dev@example.com"),
  ]);
  tokens = { meera, arjun, dev };

  const projectsRes = await fetch(`${BASE_URL}/api/projects`, {
    headers: { Authorization: `Bearer ${meera}` },
  });
  const { projects } = (await projectsRes.json()) as {
    projects: { id: string; name: string }[];
  };
  projectId = projects.find((p) => p.name === "Q3 Launch")!.id;

  const tasksRes = await fetch(`${BASE_URL}/api/projects/${projectId}/tasks`, {
    headers: { Authorization: `Bearer ${meera}` },
  });
  const { tasks } = (await tasksRes.json()) as { tasks: { id: string }[] };
  taskId = tasks[0].id;
});

describe("task access control", () => {
  // Test A
  it("a viewer cannot update a task", async () => {
    const res = await fetch(`${BASE_URL}/api/tasks/${taskId}`, {
      method: "PATCH",
      headers: {
        Authorization: `Bearer ${tokens.dev}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title: "viewer update attempt" }),
    });
    expect(res.status).toBe(403);//its a bug, test are correct
  });

  // Test B
  it("a viewer cannot create a task", async () => {
    const res = await fetch(`${BASE_URL}/api/projects/${projectId}/tasks`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${tokens.dev}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title: "viewer create attempt" }),
    });
    expect(res.status).toBe(403);//assertion to be 403, not 401
  });

  // Test C
  it("a member can create a task", async () => {
    const res = await fetch(`${BASE_URL}/api/projects/${projectId}/tasks`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${tokens.arjun}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title: "member create — baseline" }),
    });
    expect(res.status).toBe(201);
  });
//Test D
it("a member can update a task", async () => {
    const res = await fetch(`${BASE_URL}/api/tasks/${taskId}`, {
      method: "PATCH",
      headers: {
        Authorization: `Bearer ${tokens.arjun}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ title: "member update — baseline" }),
    });
    expect(res.status).toBe(200);
});
})



 FAIL  src/tests/part2.test.ts > task access control > a viewer cannot update a task
AssertionError: expected 200 to be 403 // Object.is equality

- Expected
+ Received

- 403
+ 200
Testrun output Test Files  1 failed (1)
      Tests  1 failed | 3 passed (4)
   Start at  10:37:33
   Duration  15.62s (transform 102ms, setup 2.01s, collect 37ms, tests 11.17s, environment 0ms, prepare 1.13s)

