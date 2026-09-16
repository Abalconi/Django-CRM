<script>
  import { resolve } from '$app/paths';
  import { asInternalPath } from '$lib/utils/paths.js';
  import { page } from '$app/state';
  import {
    Sun,
    Columns3,
    Target,
    Building2,
    Users,
    CircleCheck,
    LifeBuoy,
    BookOpen,
    Receipt,
    Trophy,
    Clock,
    UserCog,
    CircleUser,
    CircleHelp,
    FileText,
    Bell,
    SlidersHorizontal,
    Search,
    Smartphone,
    LogOut
  } from '@lucide/svelte';
  import { t } from '$lib/terminology.js';
  import { tr } from '$lib/i18n.js';
  import LanguageSelector from '$lib/components/LanguageSelector.svelte';

  /**
   * One flat tree, grouped by what the person is doing rather than by which
   * Django app owns the model. Every label matches the route it lands on and
   * the page title it lands on; "Pipeline" goes to /v2/pipeline, which is
   * titled "Pipeline".
   *
   * v1 had /leads listed twice, as "Pipeline" and as "Leads", and a "Deals"
   * entry pointing at /opportunities while /deals 404'd.
   *
   * `role` is server-derived from the JWT (see the app layout loader). It only
   * decides which destinations to *show*. Every hidden one is still enforced
   * by the backend, so this is UX, not access control. An item marked `admin`
   * is one where a member gets nothing but a "for administrators" gate, so
   * showing it would only teach them to bounce off it.
   *
   * `termKey` marks the handful of entity destinations a vertical pack may
   * relabel (see `$lib/terminology.js`). The string in `label` below is only
   * ever the fallback an org with no pack, or no override for that key,
   * still renders; the derived `groups` below is what actually resolves it
   * against `terminology`. No other label branches on the org at all.
   *
   * @type {{
   *   counts?: Record<string, number>,
   *   org?: { name: string },
   *   role?: string,
   *   terminology?: Record<string, string> | null,
  *   language?: string,
   *   onsearch?: () => void
   * }}
   */
  let {
    counts = {},
    org = { name: 'BottleCRM' },
    role = 'USER',
    terminology = undefined,
    language = 'es',
    onsearch = () => {}
  } = $props();

  const GROUPS = [
    {
      key: 'sell',
      items: [
        { href: '/', label: 'Today', key: 'today', icon: Sun, exact: true },
        {
          href: '/pipeline',
          label: 'Pipeline',
          key: 'pipeline',
          icon: Columns3,
          count: 'pipeline',
          termKey: 'opportunity.plural'
        },
        { href: '/leads', label: 'Leads', key: 'leads', icon: Target, count: 'leads', termKey: 'lead.plural' },
        { href: '/accounts', label: 'Accounts', key: 'accounts', icon: Building2, termKey: 'account.plural' },
        { href: '/contacts', label: 'Contacts', key: 'contacts', icon: Users, termKey: 'contact.plural' },
        { href: '/goals', label: 'Goals', key: 'goals', icon: Trophy }
      ]
    },
    {
      key: 'serve',
      items: [
        { href: '/tasks', label: 'Tasks', key: 'tasks', icon: CircleCheck, count: 'tasks' },
        // Approvals and Analytics live under Tickets as section tabs. They are
        // not separate destinations, so they do not get separate nav entries,
        // one level of navigation, and the tab strip carries the rest.
        { href: '/tickets', label: 'Tickets', key: 'tickets', icon: LifeBuoy, count: 'tickets' },
        { href: '/solutions', label: 'Knowledge base', key: 'knowledgeBase', icon: BookOpen },
        { href: '/documents', label: 'Documents', key: 'documents', icon: FileText }
      ]
    },
    {
      key: 'bill',
      items: [
        {
          href: '/invoices',
          label: 'Invoices',
          key: 'invoices',
          icon: Receipt,
          count: 'invoices',
          termKey: 'invoice.plural'
        },
        { href: '/timesheet', label: 'Timesheet', key: 'timesheet', icon: Clock }
      ]
    },
    {
      // Administration, kept apart from the work. Someone who never touches
      // these should not read past them four times a day.
      //
      // Team is admin-only. A member reaches it only to be told so. Settings
      // is not: the hub is readable by any member (it just omits admin-only
      // counts), so it stays for everyone.
      key: 'run',
      items: [
        { href: '/team', label: 'Team and access', key: 'teamAccess', icon: UserCog, admin: true },
        { href: '/settings', label: 'Settings', key: 'settings', icon: SlidersHorizontal }
      ]
    }
  ];

  // Drop admin-only items for members, resolve any relabelled entity through
  // the terminology map, then drop any group left with nothing.
  let groups = $derived(
    GROUPS.map((group) => ({
      ...group,
      label: tr(language, group.key),
      items: group.items
        .filter((item) => role === 'ADMIN' || !item.admin)
        .map((item) => ({
            ...item,
            label: item.termKey
              ? t(terminology, item.termKey, tr(language, item.key))
              : tr(language, item.key)
          }))
    })).filter((group) => group.items.length > 0)
  );

  const isActive = (href, exact) =>
    exact ? page.url.pathname === href : page.url.pathname.startsWith(href);
</script>

<nav class="v2-nav" aria-label="Main">
  <div class="v2-org">
    <span class="v2-mark">{org.name.slice(0, 1)}</span>
    <b>{org.name}</b>
  </div>
  <LanguageSelector />

  <!--
    No entry appears here without a route behind it. v1's "Deals" pointed at
    /opportunities while /deals 404'd; an Inbox link with nothing behind it
    would be the same mistake.
  -->
  {#each groups as group (group.label)}
    <div class="v2-nav-group v2-label">{group.label}</div>
    {#each group.items as item (item.href)}
      <a
        class="v2-link"
        href={resolve(asInternalPath(item.href))}
        aria-current={isActive(item.href, item.exact) ? 'page' : undefined}
      >
        <item.icon />
        {item.label}
        {#if item.count && counts[item.count]}
          <span class="v2-count">{counts[item.count]}</span>
        {/if}
      </a>
    {/each}
  {/each}

  <div class="v2-nav-foot">
    <button class="v2-link v2-nav-search" type="button" onclick={onsearch}>
      <Search />
      {tr(language, 'search')}
      <span class="v2-count">⌘K</span>
    </button>
    <!-- Personal, not work: your own feed sits with your own profile rather
         than in Serve, where it would read as a queue the team shares. -->
    <a
      class="v2-link"
      href={resolve('/notifications')}
      aria-current={isActive('/notifications', false) ? 'page' : undefined}
    >
      <Bell />
      {tr(language, 'notifications')}
      {#if counts.notifications}
        <span class="v2-count">{counts.notifications}</span>
      {/if}
    </a>
    <a class="v2-link" href={resolve('/profile')}>
      <CircleUser />
      {tr(language, 'profile')}
    </a>
    <a class="v2-link" href={resolve('/help')}>
      <CircleHelp />
      {tr(language, 'help')}
    </a>
    <!-- The phone app for people on the hosted service. No pulsing dot. A
         download link is not something that needs you right now, and v2 keeps
         attention for the things that do. -->
    <a
      class="v2-link"
      href="https://play.google.com/store/apps/details?id=io.bottlecrm&hl=en"
      target="_blank"
      rel="noopener noreferrer"
    >
      <Smartphone />
      {tr(language, 'downloadApp')}
    </a>
    <!-- Leaving the app. Last in the list, and a plain link. /logout is a
         server load that clears the auth cookies and redirects to /login, so a
         GET navigation is all it takes and no data-fetching component follows. -->
    <a class="v2-link" href={resolve('/logout')} data-sveltekit-reload>
      <LogOut />
      {tr(language, 'signOut')}
    </a>
  </div>
</nav>

<style>
  /* Search opens an overlay rather than navigating, so it is a button. It
     borrows .v2-link for everything else. A control that sits in a list of
     links should not look like the odd one out. */
  .v2-nav-search {
    width: 100%;
    background: none;
    border: 0;
    font-family: inherit;
    font-size: inherit;
    text-align: left;
    cursor: pointer;
  }
</style>
