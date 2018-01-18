class Field(object):

    _slots = {
        'args': EMPTY_DICT,             # the parameters given to __init__()
        '_attrs': EMPTY_DICT,           # the field's non-slot attributes
        '_module': None,                # the field's module name
        '_setup_done': None,            # the field's setup state: None, 'base' or 'full'

        'automatic': False,             # whether the field is automatically created ("magic" field)
        'inherited': False,             # whether the field is inherited (_inherits)

        'name': None,                   # name of the field
        'model_name': None,             # name of the model of this field
        'comodel_name': None,           # name of the model of values (if relational)

        'store': True,                  # whether the field is stored in database
        'index': False,                 # whether the field is indexed in database
        'manual': False,                # whether the field is a custom field
        'copy': True,                   # whether the field is copied over by BaseModel.copy()
        'depends': (),                  # collection of field dependencies
        'recursive': False,             # whether self depends on itself
        'compute': None,                # compute(recs) computes field on recs
        'compute_sudo': False,          # whether field should be recomputed as admin
        'inverse': None,                # inverse(recs) inverses field on recs
        'search': None,                 # search(recs, operator, value) searches on self
        'related': None,                # sequence of field names, for related fields
        'related_sudo': True,           # whether related fields should be read as admin
        'company_dependent': False,     # whether ``self`` is company-dependent (property field)
        'sparse': None,                 # the name of the corresponding serialized field, if any
        'default': None,                # default(recs) returns the default value

        'string': None,                 # field label
        'help': None,                   # field tooltip
        'readonly': False,              # whether the field is readonly
        'required': False,              # whether the field is required
        'states': None,                 # set readonly and required depending on state
        'groups': None,                 # csv list of group xml ids
        'change_default': False,        # whether the field may trigger a "user-onchange"
        'deprecated': None,             # whether the field is deprecated

        'related_field': None,          # corresponding related field
        'group_operator': None,         # operator for aggregating values
        'group_expand': None,           # name of method to expand groups in read_group()
        'prefetch': True,               # whether the field is prefetched
    }
