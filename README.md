# Socker

Socker is a library for [Go](https://golang.org) to simplify the use of
SSH, inspired by [Fabric](http://www.fabfile.org) for Python.

# Documentation
Documentation can be found at [Godoc](https://godoc.org/github.com/cosiner/socker)

# Example
```Go
const (
	ADDR_AGENT_FOO = "127.0.0.1:22"
	ADDR_AGENT_BAR = ADDR_AGENT_FOO
	ADDR_GATE      = ADDR_AGENT_FOO
)

var (
	auth = MuxAuth{
		Default: &Auth{
			User:           "root",
			PrivateKeyFile: "/home/user/.ssh/id_rsa",
		},
		Agents: map[string]*Auth{
			ADDR_GATE: {
				User:           "root",
				PrivateKeyFile: "/home/user/.ssh/gate",
			},
		},
	}
)

func TestSSH(t *testing.T) {
	testSSH(t, nil)
}

func TestGate(t *testing.T) {
	gate, err := Dial(ADDR_GATE, auth.Agents[ADDR_GATE].MustSSHConfig())
	if err != nil {
		t.Fatal(err)
	}
	defer gate.Close()

	testSSH(t, gate)
}

func testSSH(t *testing.T, gate *SSH) {
	agent, err := Dial(ADDR_AGENT_FOO, auth.Default.MustSSHConfig(), gate)
	if err != nil {
		t.Fatal(err)
	}
	defer agent.Close()

	output, err := agent.Rcmd("ls ~/")
	_ = output
	_ = err

	_ = agent.Get("~/remote", "~/local")
	_ = agent.Put("~/local", "~/remote")
}

func TestMux(t *testing.T) {
	mux, err := NewMux(auth)
	if err != nil {
		t.Fatal(err)
	}
	defer mux.Close()
	mux.Keepalive(time.Second * 10)

	agentFoo, err := mux.Dial(ADDR_AGENT_FOO, ADDR_GATE)
	if err != nil {
		t.Error(err)
		return
	}
	defer agentFoo.Close()

	agentBar, err := mux.Dial(ADDR_AGENT_BAR, "")
	if err != nil {
		t.Error(err)
		return
	}
	defer agentBar.Close()
}
```

# LICENSE
MIT.
