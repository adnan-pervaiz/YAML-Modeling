# YAML-Modeling

---
  - name: Capturing pre post activity logs
    hosts: test
    gather_facts: false
    connection: network_cli

    vars_files:
       - vars/creds_model.yml
    
    vars:
    
      ips: "{%- if ansible_host == 'RCEXY91303' -%}
    
                      10.202.1.206
    
            {%- elif  ansible_host == 'RCEXY91304' -%}
    
                      10.202.1.58
    
            {%- elif  ansible_host == 'RCKNY91001' -%}
    
                      149.135.159.2
    
            {%- elif  ansible_host == 'RCKNY91002' -%}
    
                      10.202.1.134
    
            {%- endif -%}"


    tasks:
# Please run the below commands on the following devices and log output to "<device>-filename.log"
# RJEXY91314 & RJWIY90514
      - name: Juniper CIR logs
        junos_command:
             commands:
                - set cli screen-length 0
                - show interfaces terse
                - show configuration | display set
                - show route instance detail
                - show route summary
                - show bgp summary | match Estab
                - show bgp summary | match Estab | count
                - show chassis routing-engine
                - show chassis fpc
                - show pfe statistics traffic
                - show chassis fpc
                - show services sessions count 
                - show services policies hit-count
                - show services sessions utilization
                - show interfaces load-balancing detail
                - show bgp summary instance INTER_SIG_OUTSIDE | match 10.202.1.206
                - show route advertising-protocol bgp 10.202.1.206 table INTER_SIG_OUTSIDE | count
                - show route advertising-protocol bgp 10.202.1.206 table INTER_SIG_OUTSIDE
                - show route receive-protocol bgp 10.202.1.206 table INTER_SIG_OUTSIDE | count
                - set cli screen-length 40
        register: cir_cfg
        when: ansible_host =='RJEXY91314' or 
              ansible_host =='RJWIY90514'

      - name: save output to file
        copy:
          content: "{{ cir_cfg.stdout | replace('\\n', '\n') }}"
          dest: "{{ inventory_hostname }}-{{ filename }}.log"

# Please run the below commands on the following devices and log output to "<device>-filename.log"
# RCEXY91303, RCEXY91304, RCKNY91001 & RCKNY91002
      - name: Cisco GRX logs
        ios_command:
           commands:
              - terminal length 0
              - show running-config
              - show ip interface brief
              - show vlan brief
              - show ip vrf brief
              - show standby brief
              - show etherchannel summary
              - show ip bgp vpnv4 all summary
              - show spanning-tree
              - show ip ospf neighbor
              - show cdp neighbors
              - show environment alarm
              - show ip bgp vpnv4 vrf GRX_VRF summary 
              - show ip bgp vpnv4 vrf INTER_SIG summary 
              - show ip bgp vpnv4 vrf TELSTRA_SIGNALING summary
              - show ip route vrf GRX_VRF
              - show ip route vrf INTER_SIG
              - show ip route vrf TELSTRA_SIGNALING
              - show ip bgp vpnv4 vrf INTER_SIG neighbors {{ ips }} routes
              - show ip bgp vpnv4 vrf INTER_SIG neighbors {{ ips }} advertised-routes
              - show ip bgp vpnv4 vrf INTER_SIG neighbors {{ ips }} routes | i Total
              - show ip bgp vpnv4 vrf INTER_SIG neighbors {{ ips }} advertised-routes | i Total
              - terminal length 40
        register: grx_cfg
        when: ansible_host =='RCEXY91303' or 
              ansible_host =='RCEXY91304' or
              ansible_host == 'RCKNY91001' or
              ansible_host == 'RCKNY91002'

      - name: save output to file
        copy:
          content: "{{ grx_cfg.stdout | replace('\\n', '\n') }}"
          dest: "{{ inventory_hostname }}-{{ filename }}.log"

# Please run the below commands on the following devices and log output to "<device>-filename.log"
# rdnvej300exh0630 , rdnvej160pee0330
      - name: RDN PE logs
        hosts: rdnpe
        junos_command:
             commands:
                - show route table S1000866R
        register: rdnpe_cfg
        when: ansible_host =='rdnvej300exh0630' or 
              ansible_host =='rdnvej160pee0330'

      - name: save output to file
        copy:
          content: "{{ rdnpe_cfg.stdout | replace('\\n', '\n') }}"
          dest: "{{ inventory_hostname }}-{{ filename }}.log"
