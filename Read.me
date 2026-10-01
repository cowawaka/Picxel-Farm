-- 1. Create Core Masterlist and Farm Saves Tables
create table if not exists public.masterlist (
    email text primary key,
    status text not null default 'Active'
);

create table if not exists public.farm_saves (
    userid text primary key,
    game_data jsonb not null,
    updated_at timestamptz not null default now()
);

create table if not exists public.market_listings (
    listing_id text primary key,
    created_at timestamptz not null default now(),
    seller_email text not null,
    seller_name text not null,
    item text not null,
    metadata jsonb,
    price numeric not null,
    description text not null,
    status text not null default 'Active'
);

alter table public.market_listings add column if not exists metadata jsonb;

create table if not exists public.chat_messages (
    id uuid primary key default gen_random_uuid(),
    created_at timestamptz not null default now(),
    sender_email text not null,
    recipient_email text,
    message text not null check (length(btrim(message)) between 1 and 500)
);

create table if not exists public.farm_showcases (
    id uuid primary key default gen_random_uuid(),
    user_id text not null,
    farm_name text not null,
    image_url text not null,
    storage_path text not null,
    created_at timestamptz not null default now()
);

-- 2. Legacy Farm Save Function (Updated to carry forward pixelFarmState)
create or replace function public.save_farm_state(
    p_userid text,
    p_game_data jsonb,
    p_coin_delta numeric
) returns jsonb
language plpgsql
security definer
set search_path = public, pg_temp
as $$
declare
    v_userid text := lower(trim(p_userid));
    v_current jsonb;
    v_coins numeric;
    v_next jsonb;
begin
    if v_userid = '' or jsonb_typeof(p_game_data) <> 'object' then
        return jsonb_build_object('success', false, 'message', 'Invalid farm save.');
    end if;

    select game_data into v_current from public.farm_saves where userid = v_userid for update;
    if not found then
        v_coins := coalesce((p_game_data #>> '{stats,coins}')::numeric, 0);
        insert into public.farm_saves(userid, game_data, updated_at)
        values (v_userid, jsonb_set(p_game_data, '{stats,coins}', to_jsonb(v_coins), true), now())
        on conflict (userid) do nothing;
        select game_data into v_current from public.farm_saves where userid = v_userid for update;
    end if;

    v_coins := coalesce((v_current #>> '{stats,coins}')::numeric, 0) + coalesce(p_coin_delta, 0);
    v_next := jsonb_set(
        p_game_data,
        '{stats,coins}',
        to_jsonb(v_coins),
        true
    );
    if v_current ? 'pixelFarmState' and not (p_game_data ? 'pixelFarmState') then
        v_next := jsonb_set(v_next, '{pixelFarmState}', v_current->'pixelFarmState', true);
    end if;
    update public.farm_saves set game_data = v_next, updated_at = now() where userid = v_userid;
    return jsonb_build_object('success', true, 'coins', v_coins);
end;
$$;

-- 3. Pixel Farm RPC Load Function
create or replace function public.load_pixel_farm_state(p_userid text)
returns jsonb
language plpgsql
security definer
set search_path = public, pg_temp
as $$
declare
    v_userid text := lower(trim(p_userid));
    v_state jsonb;
begin
    if not exists (
        select 1 from public.masterlist
        where lower(trim(email)) = v_userid and status = 'Active'
    ) then
        raise exception 'Farm account is not active.';
    end if;

    select game_data->'pixelFarmState' into v_state
    from public.farm_saves where userid = v_userid;
    return v_state;
end;
$$;

-- 4. Pixel Farm RPC Save Function
create or replace function public.save_pixel_farm_state(
    p_userid text,
    p_pixel_farm_data jsonb
) returns jsonb
language plpgsql
security definer
set search_path = public, pg_temp
as $$
declare
    v_userid text := lower(trim(p_userid));
    v_grid_size integer;
begin
     if v_userid = '' or jsonb_typeof(p_pixel_farm_data) is distinct from 'object'
         or p_pixel_farm_data->>'schemaVersion' is distinct from '1'
         or jsonb_typeof(p_pixel_farm_data->'gridSize') is distinct from 'number'
         or (p_pixel_farm_data->>'gridSize') !~ '^[0-9]+$'
         or jsonb_typeof(p_pixel_farm_data->'grid') is distinct from 'array'
         or jsonb_typeof(p_pixel_farm_data->'crops') is distinct from 'array'
         or jsonb_typeof(p_pixel_farm_data->'buildings') is distinct from 'array'
         or jsonb_typeof(p_pixel_farm_data->'decorations') is distinct from 'array'
         or jsonb_typeof(p_pixel_farm_data->'animals') is distinct from 'array'
       or octet_length(p_pixel_farm_data::text) > 250000 then
        raise exception 'Invalid pixel-farm save.';
    end if;

    v_grid_size := (p_pixel_farm_data->>'gridSize')::integer;
    if v_grid_size < 12 or v_grid_size > 48
       or jsonb_array_length(p_pixel_farm_data->'grid') <> v_grid_size
       or exists (
           select 1 from jsonb_array_elements(p_pixel_farm_data->'grid') as grid_row(value)
              where jsonb_typeof(grid_row.value) is distinct from 'array'
                  or case when jsonb_typeof(grid_row.value) = 'array'
                             then jsonb_array_length(grid_row.value)
                             else -1
                      end <> v_grid_size
       ) then
        raise exception 'Pixel-farm grid dimensions do not match.';
    end if;

    if not exists (
        select 1 from public.masterlist
        where lower(trim(email)) = v_userid and status = 'Active'
    ) then
        raise exception 'Farm account is not active.';
    end if;

    insert into public.farm_saves(userid, game_data, updated_at)
    values (v_userid, jsonb_build_object('pixelFarmState', p_pixel_farm_data), now())
    on conflict (userid) do update
    set game_data = jsonb_set(
        coalesce(public.farm_saves.game_data, '{}'::jsonb),
        '{pixelFarmState}',
        p_pixel_farm_data,
        true
    ), updated_at = now();

    return jsonb_build_object('success', true, 'state', p_pixel_farm_data);
end;
$$;

-- 5. Market Place RPC Functions
create or replace function public.create_market_listing(
    p_seller_email text,
    p_seller_name text,
    p_item text,
    p_price numeric,
    p_description text,
    p_metadata jsonb default null
) returns jsonb
language plpgsql
security definer
set search_path = public, pg_temp
as $$
declare
    v_seller_email text := lower(trim(p_seller_email));
    v_seller_state jsonb;
    v_stats jsonb;
    v_inventory jsonb;
    v_decorations jsonb := '[]'::jsonb;
    v_animals jsonb := '[]'::jsonb;
    v_entry jsonb;
    v_type text;
    v_key text;
    v_count numeric;
    v_removed boolean := false;
    v_item text;
    v_listing_id text := gen_random_uuid()::text;
begin
    if v_seller_email = '' or coalesce(p_price, 0) < 1 or coalesce(trim(p_description), '') = '' then
        return jsonb_build_object('success', false, 'message', 'Invalid listing details.');
    end if;

    select game_data into v_seller_state from public.farm_saves where userid = v_seller_email for update;
    if not found then
        return jsonb_build_object('success', false, 'message', 'Seller farm save not found.');
    end if;
    v_stats := coalesce(v_seller_state->'stats', '{}'::jsonb);
    v_item := trim(p_item);

    if v_item like '% (Decor)' then
        v_type := replace(v_item, ' (Decor)', '');
        for v_entry in select value from jsonb_array_elements(coalesce(v_seller_state->'decorations', '[]'::jsonb)) loop
            if not v_removed and v_entry->>'type' = v_type then
                v_removed := true;
            else
                v_decorations := v_decorations || jsonb_build_array(v_entry);
            end if;
        end loop;
        if not v_removed then return jsonb_build_object('success', false, 'message', 'That decoration is no longer on the farm.'); end if;
        v_seller_state := jsonb_set(v_seller_state, '{decorations}', v_decorations, true);
    elsif v_item like '% (Animal)' then
        v_type := 'ANIM_' || replace(v_item, ' (Animal)', '');
        for v_entry in select value from jsonb_array_elements(coalesce(v_seller_state->'animals', '[]'::jsonb)) loop
            if not v_removed and v_entry->>'type' = v_type then
                v_removed := true;
            else
                v_animals := v_animals || jsonb_build_array(v_entry);
            end if;
        end loop;
        if not v_removed then return jsonb_build_object('success', false, 'message', 'That animal is no longer on the farm.'); end if;
        v_seller_state := jsonb_set(v_seller_state, '{animals}', v_animals, true);
    else
        if v_item !~ '^[A-Z0-9_]+$' then return jsonb_build_object('success', false, 'message', 'Invalid market item.'); end if;
        v_inventory := coalesce(v_stats->'inventory', '{}'::jsonb);
        v_count := coalesce((v_inventory->>v_item)::numeric, 0);
        if v_count < 1 then return jsonb_build_object('success', false, 'message', 'Item is no longer in inventory.'); end if;
        v_inventory := jsonb_set(v_inventory, array[v_item], to_jsonb(v_count - 1), true);
        v_stats := jsonb_set(v_stats, '{inventory}', v_inventory, true);
        v_seller_state := jsonb_set(v_seller_state, '{stats}', v_stats, true);
    end if;

    update public.farm_saves set game_data = v_seller_state, updated_at = now() where userid = v_seller_email;
    insert into public.market_listings(listing_id, created_at, seller_email, seller_name, item, metadata, price, description, status)
    values (v_listing_id, now(), v_seller_email, p_seller_name, v_item, p_metadata, p_price, trim(p_description), 'Active');
    return jsonb_build_object('success', true, 'id', v_listing_id);
end;
$$;

create or replace function public.buy_market_item(
    p_listing_id text,
    p_buyer_email text
) returns jsonb
language plpgsql
security definer
set search_path = public, pg_temp
as $$
declare
    v_listing public.market_listings%rowtype;
    v_buyer_email text := lower(trim(p_buyer_email));
    v_buyer_state jsonb;
    v_seller_state jsonb;
    v_buyer_stats jsonb;
    v_seller_stats jsonb;
    v_inventory jsonb;
    v_bought_items jsonb;
    v_animals jsonb;
    v_buyer_coins numeric;
    v_seller_coins numeric;
    v_item text;
    v_type text;
    v_animal jsonb;
    v_count numeric;
begin
    select * into v_listing from public.market_listings where listing_id = p_listing_id for update;
    if not found then return jsonb_build_object('success', false, 'message', 'Item no longer exists.'); end if;
    if v_listing.status <> 'Active' then return jsonb_build_object('success', false, 'message', 'Item already sold.'); end if;
    if lower(v_listing.seller_email) = v_buyer_email then return jsonb_build_object('success', false, 'message', 'Cannot buy your own listing.'); end if;

    perform userid from public.farm_saves
    where userid in (v_buyer_email, lower(v_listing.seller_email))
    order by userid
    for update;
    select game_data into v_buyer_state from public.farm_saves where userid = v_buyer_email;
    if not found then return jsonb_build_object('success', false, 'message', 'Buyer farm save not found.'); end if;
    select game_data into v_seller_state from public.farm_saves where userid = lower(v_listing.seller_email);
    if not found then return jsonb_build_object('success', false, 'message', 'Seller farm save not found.'); end if;

    v_buyer_stats := coalesce(v_buyer_state->'stats', '{}'::jsonb);
    v_buyer_coins := coalesce((v_buyer_stats->>'coins')::numeric, 0);
    if v_buyer_coins < v_listing.price then return jsonb_build_object('success', false, 'message', 'Not enough coins.'); end if;
    v_seller_stats := coalesce(v_seller_state->'stats', '{}'::jsonb);
    v_seller_coins := coalesce((v_seller_stats->>'coins')::numeric, 0);
    v_buyer_coins := v_buyer_coins - v_listing.price;
    v_seller_coins := v_seller_coins + v_listing.price;
    v_buyer_stats := jsonb_set(v_buyer_stats, '{coins}', to_jsonb(v_buyer_coins), true);
    v_item := v_listing.item;

    if v_item like '% (Decor)' then
        v_type := replace(v_item, ' (Decor)', '');
        v_bought_items := coalesce(v_buyer_stats->'boughtItems', '[]'::jsonb) || to_jsonb(v_type);
        v_buyer_stats := jsonb_set(v_buyer_stats, '{boughtItems}', v_bought_items, true);
        if v_listing.metadata is not null and v_listing.metadata->>'type' = v_type then
            v_buyer_state := jsonb_set(v_buyer_state, '{varietyAssets}', coalesce(v_buyer_state->'varietyAssets', '[]'::jsonb) || jsonb_build_array(v_listing.metadata), true);
        end if;
    elsif v_item like '% (Animal)' then
        v_type := 'ANIM_' || replace(v_item, ' (Animal)', '');
        v_animal := jsonb_build_object('col', 1, 'row', 1, 'type', v_type, 'zone', 'farm', 'bornAt', (extract(epoch from now()) * 1000)::bigint, 'lifespanMs', 2592000000, 'happiness', 100, 'lastCareAt', (extract(epoch from now()) * 1000)::bigint);
        v_animals := coalesce(v_buyer_state->'animals', '[]'::jsonb) || jsonb_build_array(v_animal);
        v_buyer_state := jsonb_set(v_buyer_state, '{animals}', v_animals, true);
    else
        v_inventory := coalesce(v_buyer_stats->'inventory', '{}'::jsonb);
        v_count := coalesce((v_inventory->>v_item)::numeric, 0) + 1;
        v_inventory := jsonb_set(v_inventory, array[v_item], to_jsonb(v_count), true);
        v_buyer_stats := jsonb_set(v_buyer_stats, '{inventory}', v_inventory, true);
    end if;

    v_buyer_state := jsonb_set(v_buyer_state, '{stats}', v_buyer_stats, true);
    v_seller_stats := jsonb_set(v_seller_stats, '{coins}', to_jsonb(v_seller_coins), true);
    v_seller_state := jsonb_set(v_seller_state, '{stats}', v_seller_stats, true);
    update public.farm_saves set game_data = v_buyer_state, updated_at = now() where userid = v_buyer_email;
    update public.farm_saves set game_data = v_seller_state, updated_at = now() where userid = lower(v_listing.seller_email);
    update public.market_listings set status = 'Sold' where listing_id = p_listing_id;
    return jsonb_build_object(
        'success', true,
        'item', v_item,
        'price', v_listing.price,
        'metadata', v_listing.metadata,
        'buyer_coins', v_buyer_coins,
        'buyer_inventory', coalesce(v_buyer_stats->'inventory', '{}'::jsonb),
        'buyer_bought_items', coalesce(v_buyer_stats->'boughtItems', '[]'::jsonb),
        'animal', v_animal
    );
end;
$$;

-- 6. Permissions and Row Level Security
alter table public.masterlist enable row level security;
alter table public.farm_saves enable row level security;
alter table public.market_listings enable row level security;
alter table public.chat_messages enable row level security;
alter table public.farm_showcases enable row level security;

grant select on public.masterlist to anon;
grant select, insert, update on public.farm_saves to anon;
grant select, insert on public.market_listings to anon;
revoke update on public.market_listings from anon;
grant select, insert on public.chat_messages to anon;
grant select, insert on public.farm_showcases to anon;

revoke all on function public.save_farm_state(text, jsonb, numeric) from public;
revoke all on function public.create_market_listing(text, text, text, numeric, text, jsonb) from public;
revoke all on function public.buy_market_item(text, text) from public;
grant execute on function public.save_farm_state(text, jsonb, numeric) to anon;

revoke all on function public.load_pixel_farm_state(text) from public;
revoke all on function public.save_pixel_farm_state(text, jsonb) from public;
grant execute on function public.load_pixel_farm_state(text) to anon;
grant execute on function public.save_pixel_farm_state(text, jsonb) to anon;

grant execute on function public.create_market_listing(text, text, text, numeric, text, jsonb) to anon;
grant execute on function public.buy_market_item(text, text) to anon;

drop policy if exists "Browser can check whitelist" on public.masterlist;
create policy "Browser can check whitelist" on public.masterlist
    for select to anon using (true);

drop policy if exists "Browser can read farm saves" on public.farm_saves;
create policy "Browser can read farm saves" on public.farm_saves
    for select to anon using (true);
drop policy if exists "Browser can create farm saves" on public.farm_saves;
create policy "Browser can create farm saves" on public.farm_saves
    for insert to anon with check (true);
drop policy if exists "Browser can update farm saves" on public.farm_saves;
create policy "Browser can update farm saves" on public.farm_saves
    for update to anon using (true) with check (true);

drop policy if exists "Browser can read market listings" on public.market_listings;
create policy "Browser can read market listings" on public.market_listings
    for select to anon using (true);
drop policy if exists "Browser can create market listings" on public.market_listings;
create policy "Browser can create market listings" on public.market_listings
    for insert to anon with check (true);
drop policy if exists "Browser can update market listings" on public.market_listings;

drop policy if exists "Browser can read farm showcases" on public.farm_showcases;
create policy "Browser can read farm showcases" on public.farm_showcases
    for select to anon using (true);
drop policy if exists "Browser can add farm showcases" on public.farm_showcases;
create policy "Browser can add farm showcases" on public.farm_showcases
    for insert to anon with check (true);

drop policy if exists "Browser can read chat messages" on public.chat_messages;
create policy "Browser can read chat messages" on public.chat_messages
    for select to anon using (true);
drop policy if exists "Browser can send chat messages" on public.chat_messages;
create policy "Browser can send chat messages" on public.chat_messages
    for insert to anon with check (sender_email <> '' and (recipient_email is null or recipient_email <> ''));

-- 7. Realtime Publications
do $$
begin
    if not exists (select 1 from pg_publication_tables where pubname = 'supabase_realtime' and schemaname = 'public' and tablename = 'chat_messages') then
        execute 'alter publication supabase_realtime add table public.chat_messages';
    end if;
    if not exists (select 1 from pg_publication_tables where pubname = 'supabase_realtime' and schemaname = 'public' and tablename = 'market_listings') then
        execute 'alter publication supabase_realtime add table public.market_listings';
    end if;
exception
    when others then null;
end;
$$;

-- 8. Storage Setup
insert into storage.buckets (id, name, public)
values ('farm-showcases', 'farm-showcases', true)
on conflict (id) do update set public = true;

grant select, insert on storage.objects to anon;

drop policy if exists "Public can read farm showcase images" on storage.objects;
create policy "Public can read farm showcase images" on storage.objects
    for select to anon using (bucket_id = 'farm-showcases');

drop policy if exists "Browser can upload farm showcase images" on storage.objects;
create policy "Browser can upload farm showcase images" on storage.objects
    for insert to anon with check (bucket_id = 'farm-showcases');
