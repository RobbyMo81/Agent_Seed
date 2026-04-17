# The Architect of Flight

<!-- Narrative: The Architect of Flight -->
<!-- Objective: Illustrate the Per-Flight AI Agent Seed logic through allegory. -->
<!-- Status: Active -->

preface: |
  In the silent architecture of the machine, every mission begins as a "flight."
  It is not a straight line, but a crystallization of geometry and logic.

chapters:
  - title: "The Birth of the Layer"
    content: |
      Before the first byte was processed, the Architect invoked the **Layers**.
      The world was not a flat queue; it was a stack of dimensions.
      At the base, the **Context** layer hummed with the static of raw data.
      Above it, **Intent** glowed with a singular purpose, while **Policy** acted as
      the iron skin, ensuring no action defied the ancient laws of the system.
    technical_anchor: "Layer.CONTEXT, Layer.INTENT, Layer.POLICY"

  - title: "The Geometry of Thought"
    content: |
      Then came the **Shapes**. The Architect knew that a thought is only as strong
      as its structure. For a simple task, a **Pipeline** would suffice, but for
      this mission—this flight—complexity was required. A **Mesh** emerged for
      perception, while a **DAG (Directed Acyclic Graph)** was etched to prevent
      the circular madness of infinite recursion.
    technical_anchor: "Shape.MESH, Shape.DAG, Shape.PIPELINE"

  - title: "The Spark of Connection"
    content: |
      Layers and Shapes were but ghosts until the **Connections** were forged.
            The Architect drew directed edges of weighted intent. "This node **FEEDS** that one,"
      he whispered, "and this **VALIDATES** the output of the other".
      Suddenly, the **Graph** lived. It was a nervous system of **Triggers**,
      waiting for the **Threshold** of certainty to be met before the
      **Execution** layer could claim its prize.
    technical_anchor: "ConnectionType.FEEDS, ConnectionType.VALIDATES, TriggerType.THRESHOLD"

  - title: "The Void of Pure Inference"
    content: |
      The Flight reached its zenith. The reasoning callback was initiated.
      The deterministic skeleton held firm, but the model reached out into
      the unmapped territory between nodes.

            Before the void could yield its hidden structure, the **Policy Gate** stood in judgment.
            A **Human Approval** trigger hovered at the threshold, demanding proof that imagination
            would remain bound to law before any branch could descend into action.

            "I perceive the structure within the silence and etch the first DAG across the void to
            bridge the mission's intent to its final FlightRecord."

      __________________________________________________________________
      Embedded seed logic:

      ```python
      from __future__ import annotations

      """
      Per-Flight AI Agent Seed
      ========================

      Intent
      ------
      A broad, agnostic seed for agent reasoning that avoids narrow linear execution.
      It treats every task as a dynamic structure composed of:

      - Layers
      - Shapes
      - Connections / Graphs
      - Triggers

      "Per-flight obligation" here means each task run (flight) must explicitly build,
      inspect, and update its own reasoning frame before acting.

      This file is intentionally framework-light. It does not depend on any LLM SDK.
      It provides a substrate that another system can connect to a model, tools, memory,
      or governance layer.

      Core design goals
      -----------------
      1. Non-linear reasoning support via graph-based planning.
      2. Layered decomposition rather than step-only decomposition.
      3. Trigger-driven execution and re-evaluation.
      4. Broad enough to fit analysis, planning, diagnostics, orchestration, and policy.
      5. Deterministic skeleton with explicit extension points.

      Suggested integration points
      ----------------------------
      - Replace `default_reasoning_callback` with an LLM-backed planner.
      - Replace `tool_adapter` stubs with real tools or service clients.
      - Persist `FlightRecord` to SQLite, JSON, or audit logs.
      - Add governance gates before tool invocation or action commitment.
      """

      from dataclasses import dataclass, field, asdict
      from enum import Enum
      from typing import Any, Callable, Dict, Iterable, List, Optional, Set, Tuple
      import json
      import time
      import uuid


      # -----------------------------------------------------------------------------
      # Taxonomy
      # -----------------------------------------------------------------------------


      class Layer(str, Enum):
          CONTEXT = "context"
          INTENT = "intent"
          POLICY = "policy"
          PERCEPTION = "perception"
          REASONING = "reasoning"
          MEMORY = "memory"
          TOOLING = "tooling"
          EXECUTION = "execution"
          VALIDATION = "validation"
          REFLECTION = "reflection"
          OUTPUT = "output"


      class Shape(str, Enum):
          NODE = "node"
          PIPELINE = "pipeline"
          LOOP = "loop"
          TREE = "tree"
          DAG = "dag"
          MESH = "mesh"
          HUB = "hub"
          STATE_MACHINE = "state_machine"
          CONTAINER = "container"
          BRIDGE = "bridge"


      class ConnectionType(str, Enum):
          DEPENDS_ON = "depends_on"
          FEEDS = "feeds"
          CONSTRAINS = "constrains"
          VALIDATES = "validates"
          TRIGGERS = "triggers"
          REVISES = "revises"
          ESCALATES_TO = "escalates_to"
          OBSERVES = "observes"
          BLOCKS = "blocks"
          ENABLES = "enables"


      class TriggerType(str, Enum):
          USER_INPUT = "user_input"
          THRESHOLD = "threshold"
          TIMER = "timer"
          ERROR = "error"
          POLICY_GATE = "policy_gate"
          TOOL_RESULT = "tool_result"
          REFLECTION = "reflection"
          HUMAN_APPROVAL = "human_approval"
          STATE_CHANGE = "state_change"


      class NodeStatus(str, Enum):
          PENDING = "pending"
          READY = "ready"
          ACTIVE = "active"
          BLOCKED = "blocked"
          DONE = "done"
          FAILED = "failed"
          SKIPPED = "skipped"


      # -----------------------------------------------------------------------------
      # Data structures
      # -----------------------------------------------------------------------------


      @dataclass
      class Trigger:
          name: str
          trigger_type: TriggerType
          condition: str
          payload: Dict[str, Any] = field(default_factory=dict)
          active: bool = True


      @dataclass
      class Connection:
          source: str
          target: str
          connection_type: ConnectionType
          note: str = ""


      @dataclass
      class TaskNode:
          node_id: str
          title: str
          layer: Layer
          shape: Shape = Shape.NODE
          description: str = ""
          status: NodeStatus = NodeStatus.PENDING
          tags: Set[str] = field(default_factory=set)
          inputs: Dict[str, Any] = field(default_factory=dict)
          outputs: Dict[str, Any] = field(default_factory=dict)
          triggers: List[Trigger] = field(default_factory=list)
          metadata: Dict[str, Any] = field(default_factory=dict)

          def is_actionable(self) -> bool:
              return self.status in {NodeStatus.PENDING, NodeStatus.READY, NodeStatus.ACTIVE}


      @dataclass
      class FlightGraph:
          nodes: Dict[str, TaskNode] = field(default_factory=dict)
          edges: List[Connection] = field(default_factory=list)

          def add_node(self, node: TaskNode) -> None:
              self.nodes[node.node_id] = node

          def add_edge(self, edge: Connection) -> None:
              self.edges.append(edge)

          def predecessors(self, node_id: str) -> List[TaskNode]:
              sources = [e.source for e in self.edges if e.target == node_id]
              return [self.nodes[s] for s in sources if s in self.nodes]

          def successors(self, node_id: str) -> List[TaskNode]:
              targets = [e.target for e in self.edges if e.source == node_id]
              return [self.nodes[t] for t in targets if t in self.nodes]

          def ready_nodes(self) -> List[TaskNode]:
              ready: List[TaskNode] = []
              for node in self.nodes.values():
                  if node.status not in {NodeStatus.PENDING, NodeStatus.READY}:
                      continue

                  blockers = [
                      e for e in self.edges
                      if e.target == node.node_id and e.connection_type in {
                          ConnectionType.DEPENDS_ON,
                          ConnectionType.CONSTRAINS,
                          ConnectionType.BLOCKS,
                          ConnectionType.VALIDATES,
                      }
                  ]
                  blocked = False
                  for edge in blockers:
                      source_node = self.nodes.get(edge.source)
                      if source_node and source_node.status != NodeStatus.DONE:
                          blocked = True
                          break
                  if not blocked:
                      ready.append(node)
              return ready

          def summarize(self) -> Dict[str, Any]:
              return {
                  "node_count": len(self.nodes),
                  "edge_count": len(self.edges),
                  "layers": sorted({n.layer.value for n in self.nodes.values()}),
                  "shapes": sorted({n.shape.value for n in self.nodes.values()}),
              }


      @dataclass
      class FlightRecord:
          flight_id: str
          mission: str
          created_at: float
          updated_at: float
          graph: FlightGraph
          events: List[Dict[str, Any]] = field(default_factory=list)
          working_memory: Dict[str, Any] = field(default_factory=dict)
          final_output: Optional[Dict[str, Any]] = None

          def log(self, event_type: str, **data: Any) -> None:
              now = time.time()
              self.updated_at = now
              self.events.append({"ts": now, "type": event_type, **data})


      # -----------------------------------------------------------------------------
      # Agent seed
      # -----------------------------------------------------------------------------


      ReasoningCallback = Callable[[str, Dict[str, Any]], Dict[str, Any]]
      NodeExecutor = Callable[[TaskNode, FlightRecord], Dict[str, Any]]


      class PerFlightAgentSeed:
          """
          A broad agent substrate that creates a graph for every flight.

          Key rule:
          The agent is not allowed to treat a task as a single linear queue by default.
          It must first attempt to view the task as layered structure + graph + triggers.
          """

          def __init__(
              self,
              reasoning_callback: Optional[ReasoningCallback] = None,
              node_executor: Optional[NodeExecutor] = None,
          ) -> None:
              self.reasoning_callback = reasoning_callback or self.default_reasoning_callback
              self.node_executor = node_executor or self.default_node_executor

          def launch_flight(self, mission: str, context: Optional[Dict[str, Any]] = None) -> FlightRecord:
              context = context or {}
              flight = FlightRecord(
                  flight_id=str(uuid.uuid4()),
                  mission=mission,
                  created_at=time.time(),
                  updated_at=time.time(),
                  graph=FlightGraph(),
                  working_memory={"initial_context": context},
              )
              flight.log("flight_created", mission=mission, context=context)

              self._apply_per_flight_obligation(flight)
              self._seed_base_graph(flight)
              self._expand_graph_from_reasoning(flight)
              self._activate_ready_nodes(flight)

              return flight

          def run(self, mission: str, context: Optional[Dict[str, Any]] = None, max_cycles: int = 25) -> FlightRecord:
              flight = self.launch_flight(mission=mission, context=context)

              for cycle in range(max_cycles):
                  ready = [n for n in flight.graph.ready_nodes() if n.is_actionable()]
                  if not ready:
                      flight.log("run_halted", reason="no_ready_nodes", cycle=cycle)
                      break

                  progress = False
                  for node in ready:
                      if node.status == NodeStatus.PENDING:
                          node.status = NodeStatus.READY

                      result = self._execute_node(flight, node)
                      progress = progress or result.get("progress", False)

                      if result.get("request_reframe"):
                          self._reframe_from_trigger(
                              flight,
                              trigger_name="reflection_reframe",
                              trigger_type=TriggerType.REFLECTION,
                              payload=result,
                          )

                  self._activate_ready_nodes(flight)

                  if self._is_complete(flight):
                      flight.log("run_complete", cycle=cycle)
                      break

                  if not progress:
                      flight.log("run_stalled", cycle=cycle)
                      self._reframe_from_trigger(
                          flight,
                          trigger_name="stall_reframe",
                          trigger_type=TriggerType.STATE_CHANGE,
                          payload={"cycle": cycle, "reason": "stalled"},
                      )
                      self._activate_ready_nodes(flight)

              flight.final_output = self._build_output(flight)
              return flight

          def _apply_per_flight_obligation(self, flight: FlightRecord) -> None:
              obligation = {
                  "must_map_layers": True,
                  "must_identify_shapes": True,
                  "must_create_graph": True,
                  "must_define_triggers": True,
                  "must_allow_reframing": True,
                  "must_avoid_single_path_lock_in": True,
              }
              flight.working_memory["per_flight_obligation"] = obligation
              flight.log("obligation_applied", obligation=obligation)

          def _seed_base_graph(self, flight: FlightRecord) -> None:
              base_nodes = [
                  TaskNode("context_scan", "Context Scan", Layer.CONTEXT, Shape.CONTAINER),
                  TaskNode("intent_frame", "Intent Frame", Layer.INTENT, Shape.NODE),
                  TaskNode("shape_map", "Shape Map", Layer.REASONING, Shape.MESH),
                  TaskNode("graph_map", "Graph Map", Layer.REASONING, Shape.DAG),
                  TaskNode("trigger_map", "Trigger Map", Layer.REASONING, Shape.STATE_MACHINE),
                  TaskNode("policy_gate", "Policy Gate", Layer.POLICY, Shape.BRIDGE),
                  TaskNode("execution_plan", "Execution Plan", Layer.EXECUTION, Shape.PIPELINE),
                  TaskNode("validation_gate", "Validation Gate", Layer.VALIDATION, Shape.BRIDGE),
                  TaskNode("reflection", "Reflection", Layer.REFLECTION, Shape.LOOP),
                  TaskNode("output", "Output", Layer.OUTPUT, Shape.CONTAINER),
              ]

              for node in base_nodes:
                  flight.graph.add_node(node)

              edges = [
                  Connection("context_scan", "intent_frame", ConnectionType.FEEDS),
                  Connection("intent_frame", "shape_map", ConnectionType.FEEDS),
                  Connection("intent_frame", "graph_map", ConnectionType.FEEDS),
                  Connection("intent_frame", "trigger_map", ConnectionType.FEEDS),
                  Connection("shape_map", "execution_plan", ConnectionType.FEEDS),
                  Connection("graph_map", "execution_plan", ConnectionType.FEEDS),
                  Connection("trigger_map", "execution_plan", ConnectionType.CONSTRAINS),
                  Connection("policy_gate", "execution_plan", ConnectionType.VALIDATES),
                  Connection("execution_plan", "validation_gate", ConnectionType.FEEDS),
                  Connection("validation_gate", "reflection", ConnectionType.FEEDS),
                  Connection("reflection", "output", ConnectionType.FEEDS),
              ]

              for edge in edges:
                  flight.graph.add_edge(edge)

              flight.log("base_graph_seeded", graph=flight.graph.summarize())

          def _expand_graph_from_reasoning(self, flight: FlightRecord) -> None:
              prompt = (
                  "Build a non-linear task frame. "
                  "Infer candidate layers, shapes, graph branches, and triggers. "
                  "Prefer multiple plausible paths over one fixed sequence."
              )
              reasoning = self.reasoning_callback(
                  prompt,
                  {
                      "mission": flight.mission,
                      "context": flight.working_memory.get("initial_context", {}),
                      "base_graph": flight.graph.summarize(),
                  },
              )
              flight.working_memory["reasoning_seed"] = reasoning
              flight.log("reasoning_expanded", reasoning=reasoning)

              for raw in reasoning.get("extra_nodes", []):
                  node = TaskNode(
                      node_id=raw["node_id"],
                      title=raw.get("title", raw["node_id"]),
                      layer=Layer(raw.get("layer", Layer.REASONING.value)),
                      shape=Shape(raw.get("shape", Shape.NODE.value)),
                      description=raw.get("description", ""),
                      tags=set(raw.get("tags", [])),
                      metadata=raw.get("metadata", {}),
                  )
                  flight.graph.add_node(node)

              for raw in reasoning.get("extra_edges", []):
                  flight.graph.add_edge(
                      Connection(
                          source=raw["source"],
                          target=raw["target"],
                          connection_type=ConnectionType(raw.get("connection_type", ConnectionType.FEEDS.value)),
                          note=raw.get("note", ""),
                      )
                  )

              execution_plan = flight.graph.nodes.get("execution_plan")
              if execution_plan:
                  for raw in reasoning.get("triggers", []):
                      execution_plan.triggers.append(
                          Trigger(
                              name=raw["name"],
                              trigger_type=TriggerType(raw.get("trigger_type", TriggerType.STATE_CHANGE.value)),
                              condition=raw.get("condition", "always"),
                              payload=raw.get("payload", {}),
                          )
                      )

          def _activate_ready_nodes(self, flight: FlightRecord) -> None:
              for node in flight.graph.ready_nodes():
                  if node.status == NodeStatus.PENDING:
                      node.status = NodeStatus.READY
              flight.log("nodes_activated", ready=[n.node_id for n in flight.graph.ready_nodes()])

          def _execute_node(self, flight: FlightRecord, node: TaskNode) -> Dict[str, Any]:
              node.status = NodeStatus.ACTIVE
              flight.log("node_started", node=node.node_id, title=node.title)
              try:
                  result = self.node_executor(node, flight)
                  node.outputs.update(result.get("outputs", {}))
                  node.metadata.update(result.get("metadata", {}))
                  node.status = NodeStatus.DONE if not result.get("blocked") else NodeStatus.BLOCKED
                  flight.log("node_finished", node=node.node_id, result=result)
                  return result
              except Exception as exc:
                  node.status = NodeStatus.FAILED
                  flight.log("node_failed", node=node.node_id, error=str(exc))
                  return {"progress": False, "request_reframe": True, "error": str(exc)}

          def _reframe_from_trigger(
              self,
              flight: FlightRecord,
              trigger_name: str,
              trigger_type: TriggerType,
              payload: Dict[str, Any],
          ) -> None:
              flight.log(
                  "reframe_requested",
                  trigger_name=trigger_name,
                  trigger_type=trigger_type.value,
                  payload=payload,
              )

              reasoning = self.reasoning_callback(
                  "Reframe the current flight. Detect missing layers, alternate shapes, new graph branches, and new triggers.",
                  {
                      "mission": flight.mission,
                      "payload": payload,
                      "events_tail": flight.events[-10:],
                      "graph": flight.graph.summarize(),
                  },
              )
              flight.log("reframe_reasoning", reasoning=reasoning)

              for raw in reasoning.get("extra_nodes", []):
                  if raw["node_id"] not in flight.graph.nodes:
                      flight.graph.add_node(
                          TaskNode(
                              node_id=raw["node_id"],
                              title=raw.get("title", raw["node_id"]),
                              layer=Layer(raw.get("layer", Layer.REASONING.value)),
                              shape=Shape(raw.get("shape", Shape.NODE.value)),
                              description=raw.get("description", ""),
                              tags=set(raw.get("tags", [])),
                              metadata=raw.get("metadata", {}),
                          )
                      )

              for raw in reasoning.get("extra_edges", []):
                  edge = Connection(
                      source=raw["source"],
                      target=raw["target"],
                      connection_type=ConnectionType(raw.get("connection_type", ConnectionType.FEEDS.value)),
                      note=raw.get("note", ""),
                  )
                  flight.graph.add_edge(edge)

          def default_reasoning_callback(self, prompt: str, payload: Dict[str, Any]) -> Dict[str, Any]:
              mission = payload.get("mission", "")
              diagnostic_mode = any(
                  token in mission.lower() for token in ["debug", "diagnose", "issue", "failure", "error"]
              )

              extra_nodes = [
                  {
                      "node_id": "branch_hypotheses",
                      "title": "Branch Hypotheses",
                      "layer": Layer.REASONING.value,
                      "shape": Shape.TREE.value,
                      "description": "Generate multiple candidate interpretations or paths.",
                      "tags": ["non_linear", "branching"],
                  },
                  {
                      "node_id": "boundary_scan",
                      "title": "Boundary Scan",
                      "layer": Layer.PERCEPTION.value,
                      "shape": Shape.CONTAINER.value,
                      "description": "Detect environmental, interface, and trust boundaries.",
                      "tags": ["boundary", "scan"],
                  },
                  {
                      "node_id": "feedback_loop",
                      "title": "Feedback Loop",
                      "layer": Layer.REFLECTION.value,
                      "shape": Shape.LOOP.value,
                      "description": "Use observed outcomes to revise the graph.",
                      "tags": ["feedback", "revision"],
                  },
              ]

              extra_edges = [
                  {"source": "context_scan", "target": "boundary_scan", "connection_type": ConnectionType.FEEDS.value},
                  {"source": "intent_frame", "target": "branch_hypotheses", "connection_type": ConnectionType.FEEDS.value},
                  {
                      "source": "branch_hypotheses",
                      "target": "execution_plan",
                      "connection_type": ConnectionType.ENABLES.value,
                      "note": "Multiple candidate paths should inform planning.",
                  },
                  {
                      "source": "validation_gate",
                      "target": "feedback_loop",
                      "connection_type": ConnectionType.TRIGGERS.value,
                  },
                  {"source": "feedback_loop", "target": "graph_map", "connection_type": ConnectionType.REVISES.value},
              ]

              if diagnostic_mode:
                  extra_nodes.append(
                      {
                          "node_id": "fault_isolation",
                          "title": "Fault Isolation",
                          "layer": Layer.VALIDATION.value,
                          "shape": Shape.DAG.value,
                          "description": "Map candidate failure domains and isolate fault classes.",
                          "tags": ["diagnostic", "fault"],
                      }
                  )
                  extra_edges.append(
                      {"source": "boundary_scan", "target": "fault_isolation", "connection_type": ConnectionType.FEEDS.value}
                  )
                  extra_edges.append(
                      {"source": "fault_isolation", "target": "validation_gate", "connection_type": ConnectionType.VALIDATES.value}
                  )

              return {
                  "extra_nodes": extra_nodes,
                  "extra_edges": extra_edges,
                  "triggers": [
                      {
                          "name": "ambiguity_trigger",
                          "trigger_type": TriggerType.THRESHOLD.value,
                          "condition": "confidence < 0.65 or competing_paths > 1",
                          "payload": {"action": "expand_branches"},
                      },
                      {
                          "name": "error_trigger",
                          "trigger_type": TriggerType.ERROR.value,
                          "condition": "any node failed",
                          "payload": {"action": "reframe"},
                      },
                      {
                          "name": "policy_trigger",
                          "trigger_type": TriggerType.POLICY_GATE.value,
                          "condition": "proposed_action has external side effect",
                          "payload": {"action": "require_review"},
                      },
                  ],
              }

          def default_node_executor(self, node: TaskNode, flight: FlightRecord) -> Dict[str, Any]:
              mission = flight.mission

              if node.node_id == "context_scan":
                  return {
                      "progress": True,
                      "outputs": {
                          "context_summary": {
                              "mission": mission,
                              "provided_context": flight.working_memory.get("initial_context", {}),
                          }
                      },
                  }

              if node.node_id == "intent_frame":
                  return {
                      "progress": True,
                      "outputs": {"intent": {"primary_goal": mission, "mode": "broad_non_linear_frame"}},
                  }

              if node.node_id == "shape_map":
                  return {
                      "progress": True,
                      "outputs": {
                          "shape_candidates": [
                              Shape.PIPELINE.value,
                              Shape.DAG.value,
                              Shape.LOOP.value,
                              Shape.TREE.value,
                          ]
                      },
                  }

              if node.node_id == "graph_map":
                  graph_projection = {
                      "candidate_branches": [
                          "direct_execution",
                          "analysis_first",
                          "validation_first",
                          "reflective_loop",
                      ],
                      "current_nodes": list(flight.graph.nodes.keys()),
                  }
                  return {"progress": True, "outputs": {"graph_projection": graph_projection}}

              if node.node_id == "trigger_map":
                  return {
                      "progress": True,
                      "outputs": {"trigger_logic": [asdict(t) for t in flight.graph.nodes["execution_plan"].triggers]},
                  }

              if node.node_id == "policy_gate":
                  return {
                      "progress": True,
                      "outputs": {"policy_status": "pass", "notes": "No external side effects in seed mode."},
                  }

              if node.node_id == "branch_hypotheses":
                  return {
                      "progress": True,
                      "outputs": {
                          "hypotheses": [
                              "Task is best solved as a layered decomposition.",
                              "Task is best solved as graph traversal.",
                              "Task is best solved as trigger-driven state changes.",
                              "Task requires hybrid loop + DAG execution.",
                          ]
                      },
                  }

              if node.node_id == "boundary_scan":
                  return {
                      "progress": True,
                      "outputs": {
                          "boundaries": [
                              "context boundary",
                              "policy boundary",
                              "tool boundary",
                              "execution boundary",
                          ]
                      },
                  }

              if node.node_id == "execution_plan":
                  return {
                      "progress": True,
                      "outputs": {
                          "plan": {
                              "style": "non_linear",
                              "principle": "preserve alternate branches until evidence collapses them",
                              "execution_model": "graph_oriented",
                          }
                      },
                  }

              if node.node_id == "validation_gate":
                  return {
                      "progress": True,
                      "outputs": {
                          "validation": {
                              "has_layers": True,
                              "has_shapes": True,
                              "has_graph": True,
                              "has_triggers": True,
                          }
                      },
                  }

              if node.node_id == "reflection":
                  graph = flight.graph.summarize()
                  request_reframe = graph["node_count"] < 8
                  return {
                      "progress": True,
                      "request_reframe": request_reframe,
                      "outputs": {
                          "reflection": {
                              "assessment": "adequate framing" if not request_reframe else "expand graph",
                              "node_count": graph["node_count"],
                          }
                      },
                  }

              if node.node_id == "feedback_loop":
                  return {
                      "progress": True,
                      "outputs": {"feedback": "reflection can revise graph structure on subsequent passes"},
                  }

              if node.node_id == "fault_isolation":
                  return {
                      "progress": True,
                      "outputs": {
                          "fault_domains": ["inputs", "assumptions", "policy constraints", "tool coupling", "execution order"]
                      },
                  }

              if node.node_id == "output":
                  return {"progress": True, "outputs": {"output_ready": True}}

              return {"progress": True, "outputs": {"note": f"Executed generic node {node.node_id}"}}

          def _is_complete(self, flight: FlightRecord) -> bool:
              required = ["context_scan", "intent_frame", "shape_map", "graph_map", "trigger_map", "output"]
              for node_id in required:
                  node = flight.graph.nodes.get(node_id)
                  if not node or node.status != NodeStatus.DONE:
                      return False
              return True

          def _build_output(self, flight: FlightRecord) -> Dict[str, Any]:
              return {
                  "flight_id": flight.flight_id,
                  "mission": flight.mission,
                  "graph": flight.graph.summarize(),
                  "completed_nodes": [
                      node.node_id for node in flight.graph.nodes.values() if node.status == NodeStatus.DONE
                  ],
                  "event_count": len(flight.events),
                  "working_memory_keys": sorted(flight.working_memory.keys()),
              }


      def demo() -> None:
          agent = PerFlightAgentSeed()
          flight = agent.run(
              mission="Design a resilient agent workflow for a vague multi-stage task",
              context={
                  "user_preference": "reason in layers, shapes, graphs, and triggers",
                  "constraints": ["broad", "agnostic", "non-linear"],
              },
          )

          print("=== FINAL OUTPUT ===")
          print(json.dumps(flight.final_output, indent=2))
          print("\n=== GRAPH NODES ===")
          for node in flight.graph.nodes.values():
              print(f"- {node.node_id:18} | {node.layer.value:10} | {node.shape.value:13} | {node.status.value}")


      if __name__ == "__main__":
          demo()

      ```

      __________________________________________________________________

    In this silence, the **Reflection** loop evaluated the stall. It marked the moment as
    **Blocked**, not failed, and initiated a **Reframe** before the graph could collapse into
    a false certainty.

  - title: "The Final Output"
    content: |
      The graph collapsed from a thousand possibilities into a single,
      perfectly formed **FlightRecord**. The **Output** layer
      containerized the truth, and the Architect watched as the mission
      reached its destination. The flight was Done.
    technical_anchor: "FlightRecord, NodeStatus.DONE"

epilogue: "The substrate remains, waiting for the next mission pulse."
