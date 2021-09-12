---
Author: HammerLi
Date: 2021-09-05
---

# 协程池

代码很简单，比C语言的线程池类似。

```golang
package pool

import (
	"errors"
	"fmt"
	"sync"
	"sync/atomic"
	"time"
)

type StateType int64

const (
	Closed  StateType = 1
	Running StateType = 2
)

var (
	ErrInvalidCapacity  = errors.New("Non-positive capacity")
	ErrInvalidWorkerNum = errors.New("Non-positive worker number")
	ErrPoolClosed       = errors.New("Pool is closed")
	ErrScheduleTimeOut  = errors.New("Task schedule timed out")
)

type worker struct {
	id int
	p  *Manager
}

func (w *worker) run() {
	defer w.p.recoverHandler(w.id)
	for j := range w.p.jobchan {
		j.Do()
	}
}

type job func()

func (j job) Do() { j() }

type Manager struct {
	panicHandler func(r interface{})
	workerNum    int
	capacity     int
	state        *StateType
	workers      []*worker
	jobchan      chan job
	sync.Mutex
}

func NewManager(capacity int, workerNum int) (*Manager, error) {
	if capacity <= 0 {
		return nil, ErrInvalidCapacity
	}
	if workerNum <= 0 {
		return nil, ErrInvalidWorkerNum
	}

	p := &Manager{
		workerNum: workerNum,
		capacity:  capacity,
		state:     new(StateType),
		workers:   make([]*worker, workerNum),
		jobchan:   make(chan job, capacity),
	}

	for i := 0; i < workerNum; i++ {
		p.initWorker(i)
	}

	p.setState(Running)

	return p, nil
}

func (p *Manager) initWorker(id int) {
	w := &worker{id: id, p: p}
	go w.run()
	p.workers[id] = w
}

func (p *Manager) WorkerNum() int { return p.workerNum }

func (p *Manager) Capacity() int { return p.capacity }

func (p *Manager) PendingJobNum() int { return len(p.jobchan) }

func (p *Manager) State() StateType { return StateType(atomic.LoadInt64((*int64)(p.state))) }

func (p *Manager) setState(s StateType) bool {
	if p.State() == s {
		return false
	}
	atomic.StoreInt64((*int64)(p.state), (int64)(s))
	return true
}

func (p *Manager) Schedule(f job) error { return p.schedule(f, nil) }

func (p *Manager) ScheduleTimeout(f job, timeout time.Duration) error {
	return p.schedule(f, time.After(timeout))
}

func (p *Manager) schedule(f job, timeout <-chan time.Time) error {
	if p.State() == Closed {
		return ErrPoolClosed
	}
	p.Lock()
	defer p.Unlock()
	select {
	case p.jobchan <- f:
		return nil
	case <-timeout:
		return ErrScheduleTimeOut
	}
}

func (p *Manager) Close() {
	if !p.setState(Closed) {
		return // already closed
	}
	p.Lock()
	close(p.jobchan)
	p.Unlock()
}

func (p *Manager) recoverHandler(id int) {
	if r := recover(); r != nil {
		if p.panicHandler != nil {
			p.panicHandler(r)
		} else {
			p.initWorker(id) // by default re-run a worker
			fmt.Printf("Worker %d Panic: %+v. Recovered!\n", id, r)
		}
	}
}

```

