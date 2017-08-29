---
title: Java EE Events
layout: post
---

An event consists of an object and its qualifiers.
Event object is payload, data, state to be propagated to the consumer.
Event qualifiers act as filter, selector to narrow the set of events consumer observes.
Observer method acts as an event consumer.

Lets define an entity:

    class Member {

        String name;

        public Member(String name) {
            this.name = name;
        }

        public String getName() {
            return this.name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }

Define this entity as an event:

    @Inject
    Event<Member> newMemberEvent;

Fire event:

    this.newMemberEvent.fire(new Member("Any Member"));

    public void onNewMember(@Observes Member newMember) {
        this.log.warning("Any " + newMember.getName());
    }

### Filtering events (Qualifiers)

`javax.inject.Qualifier` identifies qualifies annotations.
`java.lang.annotation.Target` restricts usage.

Let Hof be certain kind of an event

    @Qualifier
    @Target({ TYPE, METHOD, PARAMETER, FIELD })
    @Retention(RUNTIME)
    public @interface Hof { }

Annotate an event with just defined qualifier

    @Inject
    @Hof
    Event<Member> MemberEventHof;

Observer to filter for an event annotated with `@Hof`

    public void onHofMember(@Observes @Hof Member member) {
        this.log.warning("Hof is" + member.getName())
    }

If we fire 2 events like:

    this.memberEvent.fire(new Member("Any Member"));
    this.memberEventHof.fire(new Member("Hof Member"));

Will output be

    Any is Any Member
    Any is Hof Member
    Hof is Hof Member


Qualifier can be added dynamically with `select( new SomeQualifier() )`

Events are processed in the same thread in which `fire()` is called.
To have it called asynchronously wait for [javaee8](https://docs.jboss.org/cdi/spec/2.0.EDR1/cdi-spec.html#firing_events_sycnronously)
